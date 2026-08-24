# Журнал архитектурных решений (ортомедия)

Формат: дата — решение — статус

## 2026-08-24

- Создан репозиторий `Yumis84/ortomediya` как рабочее пространство эталонного профиля.
- Модель: одна компания → единый список всех отзывов.
- `source` = атрибут отзыва, а не отдельный раздел.
- Темы/тональность → только Analysis-слой; публично — единый список.
- AI Summary строго отделён (FACT → EVIDENCE → ANALYSIS → AI SUMMARY).
- Общий «Рейтинг Отзовика» не рассчитывается.
- Код, дизайн и самостоятельные изменения архитектуры запрещены до отдельного задания.
- Все дальнейшие решения согласовываются с Yumis.

- **DATA MODEL v1.0** создан (`docs/DATA_MODEL.md`).
  - Сущности: Company, Review, Source, Provenance, Evidence, Analysis, AI Summary, Profile Status, Versioning.
  - Жизненный цикл отзывов (active / removed_from_source / disputed / …).
  - Status flags (не один enum).
  - Проверка 12 архитектурных вопросов — пройдена.

### Согласовано Yumis (ранее NEW DECISIONS REQUIRING YUMIS APPROVAL)

1. **Формат `otzovik_id`** — ULID.
2. **Пороги volume-флагов**:
   - `no_reviews` = 0
   - `very_low_volume` = 1–4
   - `low_volume` = 5–19
   - `medium_volume` = 20–99
   - `high_volume` = 100+
3. **`original_snapshot`** при `removed_from_source` — храним метаданные + content_hash + (по возможности) полный текст во внутреннем слое. Публично текст не показываем, если источник удалил.
4. **Публикация текста** при `removed_from_source` / `hidden` / `disputed`:
   - Публично: факт существования + статус + provenance (без полного текста, если юридически/этически нельзя).
   - Machine-readable и audit: сохраняем возможность восстановления истории.
5. **`identity_confidence`** — только во внутреннем / machine-readable слое, не в основном человеческом UI.
6. **Справочник Sources** — глобальный + usage per company.
7. **Audit trail** — versioning + минимальный event log изменений статусов отзывов и профиля (достаточно для point-in-time на первом этапе).
8. **`language`** — включаем в Review уже в v1 (nullable).
9. **`temporal_trends`** — в v1 оставляем `null` / заготовку, минимальный формат определим позже.
10. **Корень модели** — Company является корнем. Отдельный объект-обёртка `Profile` не вводим.

---

Добавлять новые записи только после согласования.
