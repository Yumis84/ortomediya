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

### Утверждено Yumis (MODERATION WORKFLOW)

1. **resolved_upheld** → moderation_status = `active`, нормальный вес, без постоянной метки «оспорен». История сохраняется. ≠ hidden.
2. **Company Statement** — отдельный объект. Не отзыв, не меняет оценку/текст, не попадает в Analysis как мнение клиента. AI Summary обязан различать. Компания не может удалить отзыв через statement.
3. **Повторное оспаривание** допускается при новых основаниях/доказательствах. Без новых обстоятельств — можно закрыть без полного рассмотрения. История сохраняется.
4. **reason_code** (минимальный набор): wrong_company, wrong_branch, duplicate, fabricated_or_suspicious, offensive_or_illegal_content, privacy_violation, factual_error, source_removed, other (с текстом).
5. **under_review** — нужен. Отзыв остаётся видимым с меткой «оспорен и находится на проверке».
6. Компания может оспорить / дать доказательства / оставить statement. Решение принимает Отзовик независимо.
7. Во время under_review влияние может быть снижено; после resolved_upheld — нормальный вес; после hidden/deleted — исключение из активного набора + пересчёт.
8. Сущности разделены: REVIEW / COMPANY STATEMENT / MODERATION CASE / SOURCE STATUS / MODERATION STATUS.

---

Добавлять новые записи только после согласования.
