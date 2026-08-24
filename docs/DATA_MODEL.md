# DATA MODEL v1.0

**Репозиторий:** Yumis84/ortomediya  
**Профиль:** ortomediya.отзыв.com  
**Статус:** проектирование (код и реализация не ведутся)  
**Дата:** 2026-08-24

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
| `otzovik_id` | string (ULID/UUID) | обязательно | служебный | Канонический внутренний идентификатор |
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
| `identity_confidence` | number (0–1) | опционально | Analysis | Уверенность в правильности entity resolution |
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
| `language` | string \| null | опционально | Analysis | |
| `review_status` | enum | обязательно | служебный | См. раздел J |
| `dispute_status` | enum \| null | опционально | служебный | |
| `removal_status` | enum \| null | опционально | служебный | |
| `original_snapshot` | object \| null | опционально | provenance | Снимок на момент сбора |
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

Отдельно от provenance конкретного отзыва.

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

- Справочник источников — глобальный.
- Provenance конкретного отзыва ссылается на `source_id` + дополнительные детали сбора.

---

## D. PROVENANCE

Происхождение каждого факта (особенно отзыва).

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

Отвечает на вопросы:
- откуда взято;
- когда опубликовано в источнике;
- когда получено нами;
- какая ссылка на оригинал;
- изменялась ли информация;
- какая версия данных;
- можно ли подтвердить;
- какой confidence.

---

## E. EVIDENCE

Слой проверяемых доказательств.  
Каждый аналитический вывод должен иметь возможность вернуться к Review.

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

Минимально достаточно. При необходимости можно добавить `first_seen` / `last_seen`.

---

## F. ANALYSIS

Производный слой. **Не является источником фактов.**

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

Каждый элемент `themes` ссылается на Evidence → supporting_review_ids → Review.

---

## G. AI SUMMARY

Соответствует OTZOVIK AI SUMMARY SPECIFICATION v1.0 (режим Balanced).

```json
{
  "summary_id": "sum_01J...",
  "company_id": "otz_...",
  "text": "В отзывах часто отмечается вежливость персонала (34 упоминания из 281). ...",
  "generated_at": "2026-08-23T18:30:00Z",
  "model": "...",
  "prompt_version": "otzovik-summary-v1.0",
  "source_data_version": "...",
  "based_on_review_count": 281,
  "based_on_source_count": 3,
  "confidence_level": "high",
  "data_status_flags": [],
  "claims": [
    {
      "text": "В отзывах часто отмечается вежливость персонала",
      "evidence_id": "ev_01J...",
      "supporting_review_ids": ["rev_001", "..."],
      "frequency": 34,
      "denominator": 281
    }
  ],
  "disclaimer": "Не является рекомендацией и не заменяет чтение оригинальных отзывов."
}
```

**Архитектурная связь (обязательна):**
```
AI Summary
  └── Claim
        └── Evidence
              └── supporting_review_ids
                    └── Review (FACT)
                          └── Source + Provenance
```

---

## H. PROFILE STATUS

**Не использовать один enum.** Состояния могут сосуществовать.

### Primary status (одно значение)

`primary_status`: `active` | `draft` | `archived` | `disputed_primary`

### Status flags (массив, независимые)

```json
"status_flags": [
  "no_reviews",
  "very_low_volume",
  "low_volume",
  "medium_volume",
  "high_volume",
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

Пример одновременных флагов:  
`["verified", "stale", "multi_branch", "data_conflict"]`

`volume_*` — взаимоисключающие (выбирается один по порогам). Остальные — независимые.

---

## I. VERSIONING

| Версия | Что изменяет |
|--------|--------------|
| `profile_version` | Структура профиля, добавление/удаление крупных блоков, смена slug, merge сущностей |
| `source_data_version` | Набор или содержимое отзывов, рейтингов источников, external_ids |
| `review_version` | Изменение конкретного отзыва (текст, статус, provenance) |
| `analysis_version` | Пересчёт тем, sentiment, evidence |
| `summary_version` | Перегенерация AI Summary |

При изменении `source_data_version` Analysis и Summary считаются потенциально устаревшими до пересчёта.

---

## J. DELETED / DISPUTED REVIEWS — жизненный цикл

`review_status`:

- `active` — публикуется
- `removed_from_source` — удалён в источнике, у нас может храниться snapshot
- `disputed` — оспорен
- `restored` — восстановлен
- `hidden` — скрыт нами (модерация / жалоба)
- `deleted` — полностью удалён (редко, с audit trail)

**Правила:**
- Provenance и audit trail сохраняются всегда.
- При `removed_from_source` / `hidden` / `deleted` оригинальный текст может не публиковаться публично (юридически/этически), но machine-readable слой и история сохраняют факт существования + статус.
- Evidence и Summary пересчитываются без исключённых отзывов (или с явной пометкой).

---

## K. RELATIONSHIPS (логическая структура)

```
Company
 ├── Branches[]
 ├── external_ids
 ├── related_entities / duplicate_candidates
 ├── Reviews[]                    ← единый список
 │     ├── source (SourceRef)
 │     ├── provenance
 │     └── (ссылки из Analysis/Evidence)
 ├── Sources[]                   ← справочник (глобальный + per-company usage)
 ├── Evidence[]
 │     └── supporting_review_ids → Reviews
 ├── Analysis
 │     └── themes → Evidence
 └── AI Summary
       └── claims → Evidence → Reviews → Source + Provenance
```

Публичный список отзывов — всегда плоский (с фильтрами).  
Группировки по source / theme / branch — только в Analysis и UI-фильтрах.

---

## L. JSON EXAMPLE (структура, не реальные данные)

```json
{
  "company": {
    "otzovik_id": "otz_01J8XEXAMPLE",
    "slug": "ortomediya",
    "name": "Ортомедия",
    "legal_name": "ООО Ортомедия",
    "aliases": ["Ортомедия Калининград", "Ортомедия"],
    "description": {
      "short": "Ортопедия, травматология и реабилитация в Калининграде",
      "full": "..."
    },
    "entity_status": "active",
    "verification_status": "unverified",
    "identity_confidence": 0.92,
    "inn": null,
    "ogrn": null,
    "website": "https://example-ortomediya.ru",
    "phones": ["+7-4012-00-00-00"],
    "primary_location": {
      "address": "г. Калининград, ул. Примерная, 1",
      "geo": { "lat": 54.71, "lon": 20.51 }
    },
    "branches": [],
    "external_ids": {
      "yandex": { "id": "123456", "confidence": 0.95, "verified_at": "2026-08-20T12:00:00Z" },
      "2gis": { "id": "789012", "confidence": 0.90, "verified_at": "2026-08-20T12:00:00Z" }
    },
    "related_entities": [],
    "duplicate_candidates": [],
    "created_at": "2026-08-19T10:00:00Z",
    "updated_at": "2026-08-23T18:30:00Z",
    "profile_version": "1.0.0",
    "primary_status": "active",
    "status_flags": ["medium_volume", "unverified"]
  },
  "reviews": [
    {
      "review_id": "rev_001",
      "company_id": "otz_01J8XEXAMPLE",
      "source": {
        "type": "yandex",
        "name": "Яндекс",
        "source_id": "src_yandex"
      },
      "source_review_id": "y_987654",
      "source_url": "https://yandex.ru/maps/org/example/123456/reviews",
      "author_hash": "a1b2c3d4",
      "rating": 5,
      "review_text": "Вежливый персонал, всё объяснили.",
      "published_at": "2025-11-12T00:00:00Z",
      "collected_at": "2026-08-20T10:15:00Z",
      "updated_at": "2026-08-20T10:15:00Z",
      "branch_id": null,
      "language": "ru",
      "review_status": "active",
      "dispute_status": null,
      "removal_status": null,
      "original_snapshot": null,
      "provenance": {
        "origin": {
          "source_id": "src_yandex",
          "source_type": "yandex",
          "source_url": "https://yandex.ru/maps/org/example/123456/reviews",
          "source_review_id": "y_987654"
        },
        "published_at": "2025-11-12T00:00:00Z",
        "collected_at": "2026-08-20T10:15:00Z",
        "collection_method": "public_page",
        "was_modified": false,
        "modification_notes": null,
        "data_version": "2026-08-20T10:15:00Z",
        "verifiable": true,
        "confidence": 0.98,
        "collector": "otzovik-v1"
      },
      "content_hash": "sha256:abc..."
    }
  ],
  "sources": [
    {
      "source_id": "src_yandex",
      "type": "yandex",
      "name": "Яндекс",
      "url": "https://yandex.ru/maps/...",
      "source_status": "active",
      "last_successful_collection": "2026-08-20T10:00:00Z",
      "last_attempt": "2026-08-23T15:00:00Z",
      "collection_status": "ok",
      "coverage_notes": null
    }
  ],
  "evidence": [
    {
      "evidence_id": "ev_01J...",
      "claim": "В отзывах часто упоминается вежливость персонала",
      "label": "вежливость персонала",
      "polarity": "positive",
      "frequency": 34,
      "denominator": 281,
      "supporting_review_ids": ["rev_001", "rev_014"],
      "source_distribution": { "yandex": 20, "2gis": 9, "google": 5 },
      "confidence": 0.87,
      "created_at": "2026-08-23T18:00:00Z",
      "analysis_version": "v1"
    }
  ],
  "analysis": {
    "analysis_id": "an_01J...",
    "company_id": "otz_01J8XEXAMPLE",
    "generated_at": "2026-08-23T18:00:00Z",
    "analysis_version": "v1",
    "source_data_version": "2026-08-23T12:00:00Z",
    "themes": ["ev_01J..."],
    "sentiment": { "positive": 0.62, "neutral": 0.21, "negative": 0.17 },
    "rating_distribution": { "1": 12, "2": 8, "3": 25, "4": 90, "5": 146 },
    "source_distribution": { "yandex": 150, "2gis": 80, "google": 51 },
    "conflicts": [],
    "suspicious_flags": []
  },
  "ai_summary": {
    "summary_id": "sum_01J...",
    "company_id": "otz_01J8XEXAMPLE",
    "text": "В отзывах часто отмечается вежливость персонала (34 упоминания из 281). ...",
    "generated_at": "2026-08-23T18:30:00Z",
    "model": "example-model",
    "prompt_version": "otzovik-summary-v1.0",
    "source_data_version": "2026-08-23T12:00:00Z",
    "based_on_review_count": 281,
    "based_on_source_count": 3,
    "confidence_level": "high",
    "data_status_flags": [],
    "claims": [
      {
        "text": "В отзывах часто отмечается вежливость персонала",
        "evidence_id": "ev_01J...",
        "supporting_review_ids": ["rev_001", "rev_014"],
        "frequency": 34,
        "denominator": 281
      }
    ],
    "disclaimer": "Не является рекомендацией и не заменяет чтение оригинальных отзывов."
  }
}
```

---

## M. ПРОВЕРКА АРХИТЕКТУРЫ

1. **Не смешали ли FACT и ANALYSIS?**  
   Нет. Review.review_text + rating + published_at = FACT. Темы, sentiment, frequencies = Analysis/Evidence.

2. **Может ли AI проверить любое утверждение Summary?**  
   Да. Claim → evidence_id → supporting_review_ids → Review + provenance.

3. **Можно ли удалить/скрыть отзыв без разрушения истории?**  
   Да. Меняется `review_status`, provenance и audit trail сохраняются.

4. **Можно ли добавить новый источник без изменения модели Review?**  
   Да. Новый `source.type` + запись в справочник Sources. Review остаётся тем же типом.

5. **Можно ли добавить новый источник отзывов без создания новой модели?**  
   Да (см. п.4).

6. **Можно ли связать отзыв с филиалом?**  
   Да, через `branch_id` (nullable).

7. **Можно ли обнаружить дубли компании?**  
   Да, через `duplicate_candidates` + `related_entities` + external_ids + aliases.

8. **Можно ли восстановить состояние профиля на прошлую дату?**  
   Частично. При наличии версионирования и snapshot’ов отзывов — да. Полный point-in-time recovery требует хранения истории изменений (закладывается через versioning + audit).

9. **Можно ли понять, откуда взялось каждое число?**  
   Да. Frequency / denominator / supporting_review_ids / source_distribution.

10. **Может ли другой AI получить полный профиль без JavaScript?**  
    Да (при реализации `/profile.json` + JSON-LD + semantic HTML). Модель это поддерживает.

11. **Не создаёт ли структура ложного общего рейтинга?**  
    Нет. Общий рейтинг отсутствует. Есть только rating_distribution и рейтинги по источникам.

12. **Не превращается ли Analysis в «новую истину»?**  
    Нет. Analysis и Evidence всегда ссылаются на Review IDs. AI Summary ссылается на Evidence. Слои разделены.

---

## NEW DECISIONS REQUIRING YUMIS APPROVAL

Следующие решения **не были явно утверждены** ранее и требуют вашего подтверждения:

1. **Формат `otzovik_id`** — ULID (рекомендую) или UUID v4.
2. **Точные пороги volume-флагов** (`very_low_volume` = 1–4, `low_volume` = 5–19, `medium` = 20–99, `high` = 100+). Можно скорректировать.
3. **Хранение `original_snapshot`** при `removed_from_source` — хранить полный текст или только метаданные + hash (юридический аспект).
4. **Политика публикации текста** при статусах `removed_from_source` / `hidden` / `disputed` — показывать ли текст публично или только факт существования + статус.
5. **`identity_confidence`** — включать ли в публичный профиль или только во внутренний/machine-readable слой.
6. **Глобальный справочник Sources** vs sources, привязанные только к компании.
7. **Глубина audit trail** для point-in-time восстановления (достаточно versioning или нужен полный event log).
8. **Поддержка `language`** на уровне Review уже в v1 или отложить.
9. **Структура `temporal_trends`** — оставляем null в v1 или сразу определяем минимальный формат.
10. **Нужен ли отдельный объект `Profile`** (обёртка над Company + status + versions) или Company уже является корнем.

Все остальные элементы модели вытекают напрямую из ранее принятых принципов и не требуют нового согласования.

---

**Документация создана. Ожидаю следующего задания.**
