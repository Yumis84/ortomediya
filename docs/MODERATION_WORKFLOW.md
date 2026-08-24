# MODERATION WORKFLOW v1.0

**Репозиторий:** Yumis84/ortomediya  
**Связан с:** REVIEW_LIFECYCLE v1.0, DATA_MODEL v1.0  
**Статус:** утверждено Yumis  
**Дата:** 2026-08-24  
**Утверждено:** 2026-08-24

---

## 1. Архитектурное правило (критично)

Не смешивать сущности:

| Сущность | Что это |
|----------|---------|
| **REVIEW** | Содержание отзыва клиента |
| **COMPANY STATEMENT** | Публичный ответ компании |
| **MODERATION CASE** | Процесс проверки |
| **SOURCE STATUS** | Состояние отзыва в исходном источнике |
| **MODERATION STATUS** | Состояние проверки Отзовиком |

Это разные объекты / состояния.

---

## 2. Статусы

### Source Status
`active` | `removed_from_source` | `unavailable`

### Moderation Status
`active` | `under_review` | `disputed` | `hidden` | `restored` | `deleted`

Пример:
```
source_status: active
moderation_status: under_review
```

Отзыв **не исчезает** из общего списка только из-за начала проверки.  
Можно показать метку: «Отзыв оспорен и находится на проверке».

---

## 3. RESOLVED_UPHELD (утверждено)

Если после проверки спор **не подтверждён** и отзыв признан допустимым:

- moderation_status → `active`
- отзыв остаётся в общем списке
- нормальный вес в Analysis
- постоянная публичная метка «оспорен» **не оставляется**
- в истории (audit trail) сохраняется факт оспаривания и результат

`resolved_upheld` ≠ `hidden`.

---

## 4. COMPANY STATEMENT (утверждено)

Компания может предоставить публичный краткий ответ / statement.

Правила:
- это **отдельный объект**, не отзыв;
- не изменяет оценку и текст отзыва;
- не попадает в Analysis как мнение клиента;
- AI Summary **обязан** отличать statement компании от отзыва клиента;
- компания через этот механизм **не может** удалить отзыв.

---

## 5. Повторное оспаривание (утверждено)

- Допускается, но не бесконечно.
- Повторный спор должен содержать **новое основание** или **новые доказательства**.
- Повторное обращение без новых обстоятельств может быть закрыто без полного повторного рассмотрения.
- История всех обращений сохраняется в audit trail.

---

## 6. Minimal reason_code (утверждено)

На первом этапе:

- `wrong_company`
- `wrong_branch`
- `duplicate`
- `fabricated_or_suspicious`
- `offensive_or_illegal_content`
- `privacy_violation`
- `factual_error`
- `source_removed`
- `other` (обязательно требует текстового объяснения)

Не создавать десятки категорий.

---

## 7. under_review (утверждено)

Статус `under_review` нужен.

Означает: спор принят и находится на рассмотрении.

Отзыв остаётся видимым в общем списке с спокойной меткой.

---

## 8. Общий принцип moderation (утверждено)

Не удалять отзыв только потому, что компании он не нравится.

Компания имеет право:
- оспорить;
- указать причину;
- предоставить доказательства;
- оставить публичный ответ (Company Statement).

Отзовик принимает решение **независимо**.

---

## 9. Влияние на Analysis и AI Summary (утверждено)

| Moderation Status | Поведение |
|-------------------|-----------|
| `under_review` | Отзыв сохраняется; статус передаётся в Analysis; влияние может быть снижено; AI Summary обязан знать о споре |
| `resolved_upheld` → `active` | Нормальный вес; Summary может быть пересчитан |
| `hidden` / `deleted` / equivalent of removed | Исключается из текущего набора активных данных; история сохраняется; Analysis и Summary пересчитываются |

Все изменения — в audit trail.

---

## 10. Минимальная структура Moderation Case

```json
{
  "dispute_id": "...",
  "review_id": "...",
  "company_id": "...",
  "initiated_by": "company" | "user" | "moderator" | "system",
  "reason_code": "wrong_company" | "wrong_branch" | "duplicate" | "fabricated_or_suspicious" | "offensive_or_illegal_content" | "privacy_violation" | "factual_error" | "source_removed" | "other",
  "reason_text": null,
  "company_statement_id": null,
  "status": "open" | "under_review" | "resolved_upheld" | "resolved_rejected" | "resolved_partial",
  "created_at": "...",
  "resolved_at": null,
  "resolution_notes": null,
  "moderator_id": null
}
```

---

## 11. Company Statement (отдельная сущность)

```json
{
  "statement_id": "...",
  "review_id": "...",
  "company_id": "...",
  "text": "...",
  "created_at": "...",
  "published": true,
  "moderation_case_id": null
}
```

Не является Review. Не влияет на rating. AI обязан различать.

---

**MODERATION WORKFLOW v1.0 утверждён.**
