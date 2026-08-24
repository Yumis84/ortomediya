# JSON SCHEMA v1.0

**Репозиторий:** Yumis84/ortomediya  
**Статус:** утверждено Yumis  
**Дата:** 2026-08-24  
**Утверждено:** 2026-08-24

---

## Принцип Public JSON

Публичный JSON = прозрачность данных.  
Internal model = полная служебная информация.

Публикуем всё, что нужно AI для проверки существенных утверждений.  
Не публикуем внутренние moderation notes, антифрод-сигналы, author_hash, служебные ID, алгоритмические параметры и персональные данные, не предназначенные для публикации.

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
    primary_location: PublicLocation | null,
    branches: PublicBranch[],
    external_ids: { [source: string]: PublicExternalId },  // упрощённо
    verification_status: "unverified" | "verified_by_company" | "verified_by_otzovik",
    entity_status: "active" | "candidate" | "merged" | "disputed" | "archived",
    created_at: string,
    updated_at: string
  },

  status: {
    primary_status: "active" | "draft" | "archived" | "disputed_primary",
    status_flags: string[],
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

  conflicts: Conflict[],                      // включено в v1

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

### PublicLocation / PublicBranch (упрощённо)
```ts
{
  branch_id?: string,                         // публичный, если есть
  name?: string,
  address: string | null,
  geo?: { lat: number, lon: number } | null,
  phones?: string[],
  hours?: string | null
}
```

### PublicExternalId (упрощённо)
```ts
{
  id: string,
  url?: string | null
  // без внутренних confidence / verified_at в публичном слое
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

### Conflict (v1)
```ts
{
  type: "address_mismatch" | "name_mismatch" | "phone_mismatch" | "possible_duplicate" | "review_count_mismatch" | "branch_mismatch" | "other",
  severity: "low" | "medium" | "high",
  description: string,
  sources: string[]
}
```

### AnalysisSummary
```ts
{
  analysis_version: string,
  source_data_version: string,
  generated_at: string,
  themes: ThemeEvidence[],
  sentiment: { positive: number, neutral: number, negative: number } | null,
  rating_distribution: { [rating: string]: number } | null,
  source_distribution: { [source: string]: number } | null,
  conflicts: Conflict[]                       // может дублировать верхний уровень или ссылаться
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
  supporting_review_ids: string[],            // 3–5 примеров (лимит 5)
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
  supporting_review_ids: string[],            // 3–5 примеров, лимит 5
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
    limit: number,                            // default 50, max 100
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
  source_url_note: string | null,             // пояснение, если прямой URL отсутствует/нестабилен
  rating: number | null,
  review_text: string | null,
  published_at: string | null,
  collected_at: string,
  branch_id: string | null,
  language: string | null,
  source_status: "active" | "removed_from_source" | "unavailable",
  moderation_status: "active" | "under_review" | "disputed" | "hidden" | "restored" | "deleted",
  provenance: ProvenancePublic,
  content_hash: string
  // author_hash — НЕ включается в публичный слой v1
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

Возвращает один `ReviewPublic`.

---

## 4. Утверждённые решения (Yumis, 2026-08-24)

1. **supporting_review_ids** — 3–5 примеров (лимит 5) в profile.json / claims / themes.
2. **source_url_note** — да, для случаев отсутствия/нестабильности прямого URL. Не заменяет source_url. URL не выдумывать.
3. **author_hash** — не включать в публичный Review v1.
4. **ExternalId / Location / Branch** — упрощённая публичная структура, без внутренних служебных полей.
5. **conflicts** — включаем в v1.
6. Public JSON = прозрачность необходимых данных; Internal model = полная служебная информация.

---

**JSON SCHEMA v1.0 утверждён.**
