# JSON SCHEMA v1.0 (черновик)

**Репозиторий:** Yumis84/ortomediya  
**Связан с:** PUBLIC_JSON_CONTRACT v1.0, DATA_MODEL v1.0  
**Статус:** проектирование (требует утверждения)  
**Дата:** 2026-08-24

Детальные поля и типы для публичных endpoints.

---

## 1. /profile.json

```ts
{
  contract_version: "1.0",                    // string, required
  generated_at: string,                       // ISO-8601, required
  data_version: string,                       // required

  profile_url: string,                        // human HTML
  canonical_json: string,                     // this document URL
  reviews_endpoint: string,                   // /reviews.json

  company: {
    otzovik_id: string,                       // ULID
    slug: string,
    name: string,
    legal_name: string | null,
    aliases: string[],
    description: {
      short: string | null,
      full: string | null
    },
    website: string | null,
    phones: string[],
    primary_location: Location | null,
    branches: Branch[],
    external_ids: { [source: string]: ExternalId },
    verification_status: "unverified" | "verified_by_company" | "verified_by_otzovik",
    entity_status: "active" | "candidate" | "merged" | "disputed" | "archived",
    created_at: string,
    updated_at: string
  },

  status: {
    primary_status: "active" | "draft" | "archived" | "disputed_primary",
    status_flags: string[],                   // volume_*, stale, data_conflict, ...
    review_counts: {
      total_collected: number,
      active: number,
      under_review: number,
      disputed: number,
      hidden: number,
      removed_from_source: number
    },
    data_freshness: {
      last_successful_collection: string | null,
      stale: boolean
    }
  },

  sources: SourceSummary[],

  analysis: AnalysisSummary | null,

  ai_summary: AISummary | null,

  company_statements: CompanyStatement[],

  versions: {
    profile_version: string,
    source_data_version: string,
    analysis_version: string | null,
    summary_version: string | null
  },

  links: {
    html: string,
    reviews: string,
    jsonld?: string,
    llms_txt?: string
  }
}
```

### SourceSummary
```ts
{
  source_id: string,
  type: "yandex" | "2gis" | "google" | "otzovik" | "other",
  name: string,
  rating: number | null,
  review_count: number,
  last_review_at: string | null,
  url: string | null,
  source_status: "active" | "temporarily_unavailable" | "deprecated"
}
```

### AnalysisSummary (сжатый)
```ts
{
  analysis_version: string,
  source_data_version: string,
  generated_at: string,
  themes: ThemeEvidence[],
  sentiment: { positive: number, neutral: number, negative: number } | null,
  rating_distribution: { [rating: string]: number } | null,
  source_distribution: { [source: string]: number } | null,
  conflicts: any[]
}
```

### ThemeEvidence
```ts
{
  evidence_id: string,
  label: string,
  polarity: "positive" | "neutral" | "negative",
  frequency: number,
  denominator: number,
  supporting_review_ids: string[],            // ограниченный набор примеров
  confidence: number
}
```

### AISummary
```ts
{
  summary_id: string,
  text: string,
  generated_at: string,
  model: string,
  prompt_version: string,
  source_data_version: string,
  based_on_review_count: number,
  based_on_source_count: number,
  confidence_level: "high" | "moderate" | "low" | "insufficient" | "conflicting",
  data_status_flags: string[],
  claims: Claim[],
  disclaimer: string,
  summary_version: string
}
```

### Claim
```ts
{
  claim: string,
  evidence_count: number,
  review_count: number,
  supporting_review_ids: string[],            // примеры
  confidence: "high" | "moderate" | "low",
  evidence_id?: string
}
```

---

## 2. /reviews.json

```ts
{
  contract_version: "1.0",
  generated_at: string,
  company_id: string,
  pagination: {
    page: number,
    limit: number,
    total: number,
    pages: number,
    next: string | null,
    prev: string | null
  },
  filters_applied: object,
  reviews: ReviewPublic[]
}
```

### ReviewPublic
```ts
{
  review_id: string,
  company_id: string,
  source: {
    type: string,
    name: string,
    source_id: string
  },
  source_review_id: string | null,
  source_url: string | null,
  rating: number | null,
  review_text: string | null,                 // null если hidden / removed_from_source и политика запрещает
  published_at: string | null,
  collected_at: string,
  branch_id: string | null,
  language: string | null,
  source_status: "active" | "removed_from_source" | "unavailable",
  moderation_status: "active" | "under_review" | "disputed" | "hidden" | "restored" | "deleted",
  provenance: ProvenancePublic,
  content_hash: string
}
```

### ProvenancePublic
```ts
{
  origin: {
    source_id: string,
    source_type: string,
    source_url: string | null,
    source_review_id: string | null
  },
  published_at: string | null,
  collected_at: string,
  collection_method: "public_page" | "api" | "manual" | "other",
  was_modified: boolean,
  verifiable: boolean,
  confidence: number
}
```

---

## 3. /reviews/{review_id}.json

Возвращает один объект `ReviewPublic` (тот же тип) +  
при необходимости дополнительные поля provenance/history (в пределах публичной политики).

---

## 4. Общие правила типов

- Все даты — ISO-8601 (UTC предпочтительно).
- `null` вместо отсутствия ключа для опциональных полей, где это важно для AI.
- Массивы никогда не `null` (пустой массив `[]`).
- Числовые рейтинги — number (не string).

---

## NEW DECISIONS REQUIRING YUMIS APPROVAL

1. Точный лимит примеров `supporting_review_ids` в Claim / ThemeEvidence (рекомендую 3–5).
2. Нужно ли поле `source_url_note` когда прямой URL отсутствует.
3. Включать ли `author_hash` в публичный ReviewPublic.
4. Формат `ExternalId` и `Location` / `Branch` — оставить как в DATA_MODEL или упростить для публичного JSON.
5. Нужен ли в profile.json блок `conflicts` уже в v1 или отложить.

---

**Черновик схемы создан. Ожидаю утверждения.**
