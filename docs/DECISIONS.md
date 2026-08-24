# Журнал архитектурных решений (ортомедия)

Формат: дата — решение — статус

## 2026-08-24

- Создан репозиторий `Yumis84/ortomediya`.
- Модель: одна компания → единый список всех отзывов.
- `source` = атрибут отзыва.
- FACT → EVIDENCE → ANALYSIS → AI SUMMARY.
- Общий «Рейтинг Отзовика» не рассчитывается.
- Код и дизайн не пишутся до отдельного задания.

Утверждённые документы:
- DATA MODEL v1.0
- REVIEW LIFECYCLE v1.0
- MODERATION WORKFLOW v1.0
- MACHINE READABLE CONTRACT v1.0
- PUBLIC JSON CONTRACT v1.0
- **JSON SCHEMA v1.0**

### Утверждено Yumis (JSON SCHEMA v1)

1. supporting_review_ids в profile.json / claims / themes — 3–5 примеров (лимит 5).
2. source_url_note — да; пояснение при отсутствии/нестабильности URL; URL не выдумывать.
3. author_hash — не включать в публичный Review v1.
4. ExternalId / Location / Branch — упрощённая публичная структура без служебных полей.
5. conflicts — включаем в v1.
6. Public JSON = прозрачность необходимых данных; Internal model = полная служебная информация.

---

Добавлять новые записи только после согласования.
