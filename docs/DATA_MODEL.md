# DATA MODEL v1.0

**Репозиторий:** Yumis84/ortomediya  
**Профиль:** ortomediya.отзыв.com  
**Статус:** проектирование (код и реализация не ведутся)  
**Дата:** 2026-08-24  
**Последнее обновление:** 2026-08-24 (решения согласованы Yumis)

Модель построена строго на принятых принципах:

- Одна компания = один профиль
- Все отзывы публикуются **одним общим списком**
- `source` — атрибут конкретного отзыва
- FACT → EVIDENCE → ANALYSIS → AI SUMMARY
- Оригинальные данные, Analysis и AI Summary никогда не смешиваются
- Общий «Рейтинг Отзовика» не рассчитывается

---

## A. COMPANY / ENTITY

### Минимальный обязательный набор (для публикации профиля)

| Поле | Тип | Обязательность | Слой | Описание |
|------|-----|----------------|------|----------|
| `otzovik_id` | string (ULID) | обязательно | служебный | Канонический внутренний идентификатор |
| `slug` | string | обязательно | FACT | Человекочитаемый идентификатор (субдомен) |
| `name` | string | обязательно | FACT | Отображаемое название |
| `entity_status` | enum | обязательно | служебный | `active` \| `candidate` \| `merged` \| `disputed` \| `archived` |
| `created_at` | datetime | обязательно | служебный | |
| `updated_at` | datetime | обязательно | служебный | |
| `profile_version` | string | обязательно | служебный | Версия структуры/содержимого профиля |

### Расширенный набор

| Поле | Тип | Обязательность | Слой | Описание |
|------|-----|----------------|------|----------|
| `legal_name` | string \| null | опционально | FACT | Юридическое название |
| `aliases` | string[] | опционально | FACT | Вариации названия |
| `description` | object | опционально | FACT | `{ short, full }` |
| `verification_status` | enum | обязательно | служебный | `unverified` \| `verified_by_company` \| `verified_by_otzovik` |
| `identity_confidence` | number (0–1) | опционально | Analysis | Уверенность в entity resolution (только machine-readable / внутренний слой) |
| `inn` | string \| null | опционально | FACT | |
| `ogrn` | string \| null | опционально | FACT | |
| `website` | string \| null | опционально | FACT | |
| `phones` | string[] | опционально | FACT | |
| `primary_location` | Location | опционально | FACT | Основной адрес |
| `branches` | Branch[] | опционально | FACT | |
| `external_ids` | map<source, ExternalId> | опционально | FACT + provenance | Связь с внешними системами |
| `related_entities` | RelatedEntity[] | опционально | Analysis / служебный | Связанные юрлица |
| `duplicate_candidates` | DuplicateCandidate[] | опционально | Analysis / служебный | Кандидаты на слияние |

**Location / Branch (упрощённо):**
```json
{
  "branch_id": "...",
  "name": "...",
  "address": "...",
  "geo": { "lat": 0, "lon": 0 },
  "phones": [],
  "hours": null,
  "external_ids": {}
}
```

**ExternalId:**
```json
{
  "id": "...",
  "confidence": 0.95,
  "verified_at": "2026-08-20T12:00:00Z",
  "url": "https://..."
}
```

---

## B. REVIEW

Один объект = один оригинальный отзыв.  
Нет отдельных типов YandexReview / GoogleReview и т.д.

### Поля

| Поле | Тип | Обязательность | Слой | Описание |
|------|-----|----------------|------|----------|
| `review_id` | string | обязательно | служебный | Внутренний ID Отзовика |
| `company_id` | string | обязательно | служебный | = otzovik_id компании |
| `source` | SourceRef | обязательно | provenance | См. ниже |
| `source_review_id` | string \| null | желательно | provenance | ID в исходной системе |
| `source_url` | string \| null | желательно | provenance | Прямая ссылка на оригинал |
| `author_hash` | string \| null | опционально | provenance | Анонимизированный автор |
| `rating` | number \| null | опционально | FACT | Обычно 1–5 |
| `review_text` | string | обязательно* | FACT | Оригинальный текст (*может быть пустым при removal) |
| `published_at` | datetime \| null | желательно | FACT | Дата публикации в источнике |
| `collected_at` | datetime | обязательно | provenance | Когда мы получили |
| `updated_at` | datetime | обязательно | служебный | |
| `branch_id` | string \| null | опционально | FACT / Analysis | Привязка к филиалу (если удалось) |
| `language` | string \| null | опционально | Analysis | Включено в v1 |
| `review_status` | enum | обязательно | служебный | См. раздел J |
| `dispute_status` | enum \| null | опционально | служебный | |
| `removal_status` | enum \| null | опционально | служебный | |
| `original_snapshot` | object \| null | опционально | provenance | Снимок (метаданные + hash; полный текст — внутренний слой) |
| `provenance` | Provenance | обязательно | provenance | Подробное происхождение |
| `content_hash` | string | обязательно | служебный | Хэш текста + ключевых полей |

**SourceRef (внутри Review):**
```json
{
  "type": "yandex" | "2gis" | "google" | "otzovik" | "other",
  "name": "Яндекс",
  "source_id": "src_yandex"
}
```

---

## C. SOURCE (справочная сущность)

Глобальный справочник + usage per company.

```json
{
  "source_id": "src_yandex",
  "type": "yandex",
  "name": "Яндекс",
  "url": "https://yandex.ru/maps/...",
  "source_status": "active" | "temporarily_unavailable" | "deprecated",
  "last_successful_collection": "2026-08-20T10:00:00Z",
  "last_attempt": "2026-08-23T15:00:00Z",
  "collection_status": "ok" | "partial" | "failed",
  "coverage_notes": "..."
}
```

---

## D. PROVENANCE

```json
{
  "origin": {
    "source_id": "src_yandex",
    "source_type": "yandex",
    "source_url": "https://...",
    "source_review_id": "123456"
  },
  "published_at": "2025-11-12T00:00:00Z",
  "collected_at": "2026-08-20T10:15:00Z",
  "collection_method": "public_page" | "api" | "manual" | "other",
  "was_modified": false,
  "modification_notes": null,
  "data_version": "2026-08-20T10:15:00Z",
  "verifiable": true,
  "confidence": 0.98,
  "collector": "otzovik-v1"
}
```

---

## E. EVIDENCE

```json
{
  "evidence_id": "ev_01J...",
  "claim": "В отзывах часто упоминается вежливость персонала",
  "label": "вежливость персонала",
  "polarity": "positive",
  "frequency": 34,
  "denominator": 281,
  "supporting_review_ids": ["rev_001", "rev_014", "..."],
  "source_distribution": {
    "yandex": 20,
    "2gis": 9,
    "google": 5
  },
  "confidence": 0.87,
  "created_at": "2026-08-23T18:00:00Z",
  "analysis_version": "..."
}
```

---

## F. ANALYSIS

```json
{
  "analysis_id": "an_01J...",
  "company_id": "otz_...",
  "generated_at": "2026-08-23T18:00:00Z",
  "analysis_version": "v1",
  "source_data_version": "...",
  "themes": [ /* Evidence[] */ ],
  "sentiment": {
    "positive": 0.62,
    "neutral": 0.21,
    "negative": 0.17
  },
  "polarity_summary": "...",
  "rating_distribution": {
    "1": 12,
    "2": 8,
    "3": 25,
    "4": 90,
    "5": 146
  },
  "temporal_trends": null,
  "source_distribution": {
    "yandex": 150,
    "2gis": 80,
    "google": 51
  },
  "branch_distribution": {},
  "conflicts": [],
  "suspicious_flags": [],
  "review_quality_flags": []
}
```

---

## G. AI SUMMARY

Соответствует OTZOVIK AI SUMMARY SPECIFICATION v1.0 (режим Balanced).

Связь: AI Summary → Claim → Evidence → supporting_review_ids → Review → Source + Provenance.

---

## H. PROFILE STATUS

**Не один enum.** Состояния могут сосуществовать.

### Primary status
`primary_status`: `active` | `draft` | `archived` | `disputed_primary`

### Status flags (независимые)

```json
"status_flags": [
  "no_reviews",          // 0
  "very_low_volume",     // 1–4
  "low_volume",          // 5–19
  "medium_volume",       // 20–99
  "high_volume",         // 100+
  "stale",
  "source_unavailable",
  "data_conflict",
  "unverified",
  "verified",
  "disputed",
  "new_company",
  "multi_branch",
  "possible_duplicate"
]
```

`volume_*` — взаимоисключающие. Остальные — независимые.

---

## I. VERSIONING

| Версия | Что изменяет |
|--------|--------------|
| `profile_version` | Структура профиля, slug, merge сущностей |
| `source_data_version` | Набор/содержимое отзывов, рейтинги источников, external_ids |
| `review_version` | Изменение конкретного отзыва |
| `analysis_version` | Пересчёт тем, sentiment, evidence |
| `summary_version` | Перегенерация AI Summary |

---

## J. DELETED / DISPUTED REVIEWS — жизненный цикл

`review_status`:

- `active`
- `removed_from_source`
- `disputed`
- `restored`
- `hidden`
- `deleted`

**Правила (согласовано):**
- Provenance и audit trail сохраняются всегда.
- При `removed_from_source` / `hidden` / `deleted` публично показываем факт + статус + provenance (без полного текста, если нельзя).
- Полный текст (если хранится) — только во внутреннем слое.
- Evidence и Summary пересчитываются без исключённых отзывов (или с пометкой).

---

## K. RELATIONSHIPS

```
Company
 ├── Branches[]
 ├── external_ids
 ├── related_entities / duplicate_candidates
 ├── Reviews[]                    ← единый список
 │     ├── source (SourceRef)
 │     ├── provenance
 │     └── (ссылки из Analysis/Evidence)
 ├── Sources[]                   ← глобальный справочник
 ├── Evidence[]
 │     └── supporting_review_ids → Reviews
 ├── Analysis
 │     └── themes → Evidence
 └── AI Summary
       └── claims → Evidence → Reviews → Source + Provenance
```

Публичный список отзывов — всегда плоский (с фильтрами).  
Группировки — только в Analysis и UI-фильтрах.

---

## L. JSON EXAMPLE

(см. предыдущую версию файла — структура без изменений, только уточнены согласованные поля)

---

## M. ПРОВЕРКА АРХИТЕКТУРЫ

Все 12 вопросов проверены положительно (см. предыдущую версию).

---

## СОГЛАСОВАННЫЕ РЕШЕНИЯ (ранее NEW DECISIONS)

Все пункты, вынесенные в NEW DECISIONS REQUIRING YUMIS APPROVAL, **согласованы Yumis** и зафиксированы в `docs/DECISIONS.md` (запись от 2026-08-24).

Кратко:

1. `otzovik_id` = ULID
2. Пороги volume зафиксированы
3–4. Политика snapshot и публикации текста при удалении/скрытии
5. `identity_confidence` — только machine-readable
6. Sources — глобальный справочник
7. Versioning + минимальный event log
8. `language` в v1
9. `temporal_trends` = null в v1
10. Company = корень модели

---

**Модель данных v1.0 считается утверждённой.**
