# PUBLIC JSON CONTRACT v1.0

**Репозиторий:** Yumis84/ortomediya  
**Статус:** утверждено Yumis  
**Дата:** 2026-08-24  
**Утверждено:** 2026-08-24

---

## 1. Каноническая цепочка проверки

```
PROFILE (/profile.json)
  ↓
AI SUMMARY
  ↓
ANALYSIS / EVIDENCE
  ↓
supporting_review_ids (примеры)
  ↓
REVIEW (/reviews/{review_id}.json или /reviews.json)
  ↓
PROVENANCE
  ↓
SOURCE
```

AI-агент должен иметь возможность пройти эту цепочку и проверить существенное утверждение.

---

## 2. Пагинация (утверждено)

Простая пагинация:

`?page=1&limit=50`

- Стандартный limit: **50**
- Максимальный limit: **100** (больше — возвращается 100)
- По умолчанию: `limit=50`
- Cursor pagination в v1 **не** используем (архитектура не должна мешать переходу позже)

Объект pagination:

```json
"pagination": {
  "page": 1,
  "limit": 50,
  "total": 281,
  "pages": 6,
  "next": "/reviews.json?page=2&limit=50",
  "prev": null
}
```

---

## 3. Endpoints

| Endpoint | Назначение |
|----------|------------|
| `/profile.json` | Канонический профиль (без полного массива отзывов) |
| `/reviews.json` | Единый список отзывов с пагинацией и фильтрами |
| `/reviews/{review_id}.json` | Один конкретный отзыв + provenance (для проверки supporting_review_ids) |

Фильтр по источнику допускается: `/reviews.json?source=yandex`  
Это фильтрация **одного общего набора**, а не отдельные разделы.

---

## 4. /profile.json — обязательные блоки

- identity (company)
- sources + source_ratings
- review_count (с разбивкой по статусам)
- data_freshness
- profile_status / verification_status
- conflicts (если есть)
- analysis (сжатый)
- ai_summary
- company_statements (если есть)
- endpoints / links
- versions (`contract_version`, `data_version`, …)

Ссылки:
- `reviews_endpoint`
- `human_profile_url`
- schema / jsonld (если есть)

**Не** помещаем полный список `supporting_review_ids` для всех утверждений.

Для каждого ключевого вывода AI Summary:
- `evidence_count`
- ограниченный набор `supporting_review_ids` (примеры)
- при необходимости ссылка на Analysis/Evidence

---

## 5. AI Summary → Evidence (утверждено)

```json
{
  "claim": "В отзывах часто упоминается отношение персонала",
  "evidence_count": 34,
  "review_count": 281,
  "supporting_review_ids": ["R123", "R187", "R241"],
  "confidence": "high"
}
```

Полный набор id доступен через `/reviews.json` или специализированный запрос.

---

## 6. Один Review = один объект

Отзыв не дублируется из-за нескольких тем.

Темы относятся к Analysis.  
`review_id` остаётся единственным.

---

## 7. Provenance (минимум в публичном Review)

- `review_id`
- `company_id`
- `source`
- `source_review_id` (если доступен)
- `source_url` (если допустимо и существует)
- `published_at` / `review_date`
- `collected_at`
- `rating`
- `text` (или null при hidden/removed)
- `language`
- `source_status`
- `moderation_status`
- `branch_id` (если определён)
- `provenance` (origin, dates, method, verifiable, confidence)

**Source URL:** не придумывать. Если стабильного URL нет — указать доступную ссылку на источник/страницу компании и явно обозначить ограничение.

---

## 8. Статусы

JSON обязан различать:

```json
{
  "source_status": "active",
  "moderation_status": "under_review"
}
```

AI не должен интерпретировать `under_review` / `disputed` как обычный подтверждённый отзыв без оговорки.

---

## 9. Никакого скрытого общего рейтинга

Запрещено поле вида `"otzovik_rating": 4.72` до утверждённой методологии.

Только раздельные `source_ratings`.

---

## 10. Версионирование

- `contract_version`: "1.0"
- `data_version` (profile / source_data)
- `summary_version` (внутри AI Summary)

---

## 11. Главный принцип

Не делать JSON сложнее необходимого.

AI должен:
1. открыть `/profile.json`;
2. понять компанию и состояние данных;
3. увидеть Summary;
4. при необходимости углубиться до конкретных отзывов и provenance.

---

**PUBLIC JSON CONTRACT v1.0 утверждён.**

Следующий этап: детальная схема полей и типов для `profile.json` и `reviews.json`.
