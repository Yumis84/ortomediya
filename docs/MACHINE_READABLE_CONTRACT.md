# MACHINE READABLE CONTRACT v1.0 (черновик)

**Репозиторий:** Yumis84/ortomediya  
**Цель:** определить, как AI-агент и другие системы получают полный структурированный профиль без JavaScript и без догадок.  
**Статус:** проектирование (требует утверждения)  
**Дата:** 2026-08-24

---

## 1. Обязательные точки доступа

| Ресурс | Назначение |
|--------|------------|
| `/` или `/index.html` | Человеческая HTML-страница + semantic HTML + JSON-LD |
| `/profile.json` | Полный machine-readable dump профиля |
| `/llms.txt` или `/ai.txt` | Краткие правила использования для LLM/агентов |
| JSON-LD в `<head>` | Schema.org (Organization, AggregateRating, Review и т.д.) |

Все ключевые данные должны быть доступны **без выполнения JavaScript**.

---

## 2. Минимальное содержимое `/profile.json`

```json
{
  "schema_version": "1.0",
  "generated_at": "ISO-8601",
  "company": { /* Company object из DATA_MODEL */ },
  "reviews": [ /* массив Review (с учётом moderation/source status) */ ],
  "sources": [ /* используемые источники */ ],
  "evidence": [ /* Evidence[] */ ],
  "analysis": { /* Analysis object */ },
  "ai_summary": { /* AI Summary object или null */ },
  "company_statements": [ /* Company Statement[] */ ],
  "status": {
    "primary_status": "...",
    "status_flags": ["..."]
  },
  "versions": {
    "profile_version": "...",
    "source_data_version": "...",
    "analysis_version": "...",
    "summary_version": "..."
  }
}
```

---

## 3. Правила фильтрации в profile.json

По умолчанию:
- reviews с moderation_status in (`active`, `under_review`, `disputed`, `restored`)
- для `removed_from_source` / `hidden` — можно включать как факты без полного текста (или отдельный флаг `include_removed=true`)

AI Summary и Analysis всегда ссылаются только на те review_id, которые присутствуют в выдаче (или явно помечены как исключённые).

---

## 4. Разделение слоёв в JSON

- `reviews[].review_text` + rating + published_at = FACT
- `evidence` + `analysis` = производные
- `ai_summary` = вторичный слой с claims → evidence_id → supporting_review_ids
- `company_statements` = отдельный массив, никогда не внутри reviews

---

## 5. JSON-LD (минимум)

- `@type`: Organization / LocalBusiness
- name, url, address, telephone
- aggregateRating (только если есть данные источников, без «Рейтинга Отзовика»)
- review[] (выборка или ссылка на полный набор)

Не выдавать агрегированный «рейтинг Отзовика».

---

## 6. llms.txt / ai.txt (черновик содержания)

- Краткое описание проекта
- Ссылка на /profile.json
- Принцип FACT → EVIDENCE → ANALYSIS → AI SUMMARY
- Запрет воспринимать AI Summary как рекомендацию
- Указание, что Company Statement ≠ отзыв клиента
- Контакт для ошибок / оспаривания

---

## NEW DECISIONS REQUIRING YUMIS APPROVAL

1. Точный путь: `/profile.json` vs `/.well-known/otzovik-profile.json` vs оба.
2. Нужна ли пагинация / cursor в profile.json при очень большом числе отзывов (или всегда полный dump на первом этапе).
3. Включать ли по умолчанию reviews со status `under_review` и `disputed` в profile.json.
4. Минимальный обязательный набор Schema.org типов.
5. Нужен ли отдельный `/reviews.json` или достаточно одного profile.json.

---

**Черновик создан. Ожидаю утверждения.**
