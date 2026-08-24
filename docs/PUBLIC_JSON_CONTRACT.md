# PUBLIC JSON CONTRACT v1.0 (черновик)

**Репозиторий:** Yumis84/ortomediya  
**Связан с:** MACHINE_READABLE_CONTRACT v1.0, DATA_MODEL v1.0  
**Статус:** проектирование (требует утверждения)  
**Дата:** 2026-08-24

Точная структура публичного machine-readable контракта и правила provenance, чтобы AI мог проверить происхождение каждого существенного утверждения.

---

## 1. /profile.json — структура

```json
{
  "schema_version": "1.0",
  "generated_at": "2026-08-24T01:00:00Z",
  "profile_url": "https://ortomediya.отзыв.com",
  "canonical_json": "https://ortomediya.отзыв.com/profile.json",
  "reviews_endpoint": "https://ortomediya.отзыв.com/reviews.json",

  "company": {
    "otzovik_id": "otz_01J...",
    "slug": "ortomediya",
    "name": "Ортомедия",
    "legal_name": null,
    "aliases": [],
    "description": { "short": "...", "full": "..." },
    "website": null,
    "phones": [],
    "primary_location": {},
    "branches": [],
    "external_ids": {},
    "verification_status": "unverified",
    "entity_status": "active",
    "created_at": "...",
    "updated_at": "..."
  },

  "status": {
    "primary_status": "active",
    "status_flags": ["medium_volume", "unverified"],
    "review_counts": {
      "total_collected": 281,
      "active": 270,
      "under_review": 3,
      "disputed": 2,
      "hidden": 4,
      "removed_from_source": 6
    },
    "data_freshness": {
      "last_successful_collection": "2026-08-20T10:00:00Z",
      "stale": false
    }
  },

  "sources": [
    {
      "source_id": "src_yandex",
      "type": "yandex",
      "name": "Яндекс",
      "rating": 4.7,
      "review_count": 150,
      "last_review_at": "2026-07-01",
      "url": "https://...",
      "source_status": "active"
    }
  ],

  "analysis": {
    "analysis_version": "v1",
    "source_data_version": "...",
    "generated_at": "...",
    "themes": [
      {
        "evidence_id": "ev_01J...",
        "label": "вежливость персонала",
        "polarity": "positive",
        "frequency": 34,
        "denominator": 270,
        "supporting_review_ids": ["rev_001", "rev_014"],
        "confidence": 0.87
      }
    ],
    "sentiment": { "positive": 0.62, "neutral": 0.21, "negative": 0.17 },
    "rating_distribution": { "1": 12, "2": 8, "3": 25, "4": 90, "5": 146 },
    "source_distribution": {},
    "conflicts": []
  },

  "ai_summary": {
    "summary_id": "sum_01J...",
    "text": "...",
    "generated_at": "...",
    "model": "...",
    "prompt_version": "otzovik-summary-v1.0",
    "source_data_version": "...",
    "based_on_review_count": 270,
    "based_on_source_count": 3,
    "confidence_level": "high",
    "data_status_flags": [],
    "claims": [
      {
        "text": "В отзывах часто отмечается вежливость персонала",
        "evidence_id": "ev_01J...",
        "supporting_review_ids": ["rev_001", "rev_014"],
        "frequency": 34,
        "denominator": 270
      }
    ],
    "disclaimer": "Не является рекомендацией и не заменяет чтение оригинальных отзывов."
  },

  "company_statements": [],

  "versions": {
    "profile_version": "1.0.0",
    "source_data_version": "2026-08-20T10:00:00Z",
    "analysis_version": "v1",
    "summary_version": "..."
  },

  "links": {
    "html": "https://ortomediya.отзыв.com",
    "reviews": "https://ortomediya.отзыв.com/reviews.json",
    "jsonld": "https://ortomediya.отзыв.com/#jsonld",
    "llms_txt": "https://ortomediya.отзыв.com/llms.txt"
  }
}
```

---

## 2. /reviews.json — структура ответа

```json
{
  "schema_version": "1.0",
  "generated_at": "...",
  "company_id": "otz_01J...",
  "page": 1,
  "limit": 50,
  "total": 270,
  "next": "https://ortomediya.отзыв.com/reviews.json?page=2&limit=50",
  "filters_applied": {},
  "reviews": [
    {
      "review_id": "rev_001",
      "source": {
        "type": "yandex",
        "name": "Яндекс",
        "source_id": "src_yandex"
      },
      "source_review_id": "y_987654",
      "source_url": "https://...",
      "rating": 5,
      "review_text": "Вежливый персонал, всё объяснили.",
      "published_at": "2025-11-12T00:00:00Z",
      "collected_at": "2026-08-20T10:15:00Z",
      "branch_id": null,
      "language": "ru",
      "source_status": "active",
      "moderation_status": "active",
      "provenance": {
        "origin": {
          "source_id": "src_yandex",
          "source_type": "yandex",
          "source_url": "https://...",
          "source_review_id": "y_987654"
        },
        "published_at": "2025-11-12T00:00:00Z",
        "collected_at": "2026-08-20T10:15:00Z",
        "collection_method": "public_page",
        "was_modified": false,
        "verifiable": true,
        "confidence": 0.98
      },
      "content_hash": "sha256:..."
    }
  ]
}
```

Для `removed_from_source` / `hidden`:
- `review_text` может быть `null` или отсутствовать;
- статус обязателен;
- provenance сохраняется.

---

## 3. Правила provenance (проверка утверждений)

Любое существенное утверждение в Analysis или AI Summary должно быть прослеживаемо:

```
Claim / Theme
  → evidence_id
    → supporting_review_ids[]
      → Review (FACT)
        → provenance.origin (source + source_url + source_review_id)
        → published_at / collected_at
        → content_hash
```

AI должен иметь возможность:
1. Взять claim из AI Summary.
2. Найти evidence_id.
3. Получить supporting_review_ids.
4. Запросить эти review_id через /reviews.json (или найти в выдаче).
5. Проверить source_url / content_hash / даты.

---

## 4. Что никогда не попадает в публичный JSON

- Внутренние moderator notes
- Персональные данные авторов сверх author_hash
- Полные тексты скрытых/удалённых отзывов (если политика запрещает)
- Служебные флаги антифрода (можно агрегированные suspicious_flags без деталей)
- identity_confidence (по ранее утверждённому решению — только внутренний слой)

---

## NEW DECISIONS REQUIRING YUMIS APPROVAL

1. Точные имена query-параметров для /reviews.json (`page`+`limit` vs cursor-based).
2. Максимальный `limit` по умолчанию и hard-limit.
3. Нужно ли в /profile.json отдавать полный массив `supporting_review_ids` для каждой темы или достаточно evidence_id + frequency (полные id — только по запросу).
4. Формат `next` (URL vs opaque cursor).
5. Нужен ли endpoint `/reviews/{review_id}.json` для точечной проверки одного отзыва.

---

**Черновик создан. Ожидаю утверждения.**
