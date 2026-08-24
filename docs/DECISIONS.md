# Журнал архитектурных решений (ортомедия)

Формат: дата — решение — статус

## 2026-08-24

- Создан репозиторий `Yumis84/ortomediya`.
- Модель: одна компания → единый список всех отзывов.
- `source` = атрибут отзыва.
- FACT → EVIDENCE → ANALYSIS → AI SUMMARY.
- Общий «Рейтинг Отзовика» не рассчитывается.
- Код и дизайн не пишутся до отдельного задания.

- **DATA MODEL v1.0** утверждён.
- **REVIEW LIFECYCLE v1.0** утверждён.
- **MODERATION WORKFLOW v1.0** утверждён.
- **MACHINE READABLE CONTRACT v1.0** утверждён.
- **PUBLIC JSON CONTRACT v1.0** утверждён.

### Утверждено Yumis (PUBLIC JSON CONTRACT)

1. Пагинация v1: `?page=&limit=` (limit по умолчанию 50, max 100). Cursor — позже.
2. В profile.json — только примеры supporting_review_ids + evidence_count; полный набор через reviews.
3. Pagination object с page/limit/total/pages/next/prev.
4. Endpoint `/reviews/{review_id}.json` обязателен для проверки цепочки.
5. Provenance минимум зафиксирован; source_url не придумывать.
6. Один review = один объект (темы не создают копии).
7. Никакого `otzovik_rating` до методологии.
8. source_status и moderation_status раздельно.
9. contract_version / data_version / summary_version.
10. Каноническая цепочка: Profile → AI Summary → Evidence → review_id → Review → Provenance → Source.

---

Добавлять новые записи только после согласования.
