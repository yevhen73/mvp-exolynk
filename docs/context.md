# Контекст MVP «Заказ покупателя» на Exolynk

## Цель
Показать на практике, что платформа Exolynk позволяет собрать управляемый процесс оформления заказа с разграничением ролей, канбаном состояний и автоматизациями (workflow, уведомления) поверх PostgreSQL.

## Ключевые роли и переходы
| Роль       | Действия / переходы |
|------------|---------------------|
| Sales      | `NEW → PICKING` (≤ 10 000 CHF), `NEW → APPROVAL` (> 10 000 CHF), `CLOSED (fail)` |
| Manager    | `APPROVAL → PICKING`, `READY → CLOSED (success)` через подтверждение оплаты |
| Logistics  | `PICKING → READY`, задаёт `ship_date`, переводит `shipment_state` |
| System     | Ставит `shipment_state=SHIPPED`, закрывает заказ (`state=CLOSED`, `status=CLOSED`, `closed_success=true/false`), вычисляет статус |

## Состояния
- **state**: `NEW`, `APPROVAL`, `PICKING`, `READY`, `CLOSED`
- **status**: `OPEN`, `EXECUTION`, `CLOSED` (рассчитывается системой)
- **shipment_state**: `PICKING`, `READY`, `SHIPPED`
- **payment_state**: `NO`, `YES`

## Модели
### `order`
`id`, `number`, `client_id`, `created_at`, `status`, `state`, `shipment_state`, `payment_state`, `amount_total`, `closed_success`, `closed_reason`, `ship_date`, `comment`, `created_by`

### `order_item`
`id`, `order_id`, `sku`, `name`, `qty`, `price`, `amount`

### `client`
Базовые данные клиента (имя, контакты). Поля именуем в `snake_case`.

## UX карточки `order_detail`
- Шапка: номер, дата создания, статусы, режим редактирования.
- Грид метаданных (тип, клиент, ответственный, отгрузка и т.п.).
- Таблица «Товары и услуги» с ограничением по высоте, колонкой `×`, добавление строк только в режиме «Изменить».
- Блок комментария и блок итогов (НДС, сумма, валюта, ручные скидки).
- CTA по ролям: «Комплектация», «Разрешить комплектацию», «Готово к отгрузке», «Подтвердить оплату», «Закрыть неуспешно».

## Workflow и действия
- `sales.to_picking`
- `manager.allow_picking`
- `logistics.ready`
- `manager.confirm_payment`
- `sales.close_fail`

### Сервисы/шаблоны
`svc_notify_manager_approval`, `svc_notify_logistics_picking`, `svc_notify_manager_payment`, `svc_close_success` + email/webhook шаблоны для уведомлений.

## Документация
- Getting started: https://exolynk.app/docs/docs/usage/getting-started/
- Rune API: https://exolynk.app/docs/docs/api/rune/
- REST API: https://exolynk.app/docs/docs/api/rest/
- Templates: https://exolynk.com/en/templates/

## Открытые вопросы
- Подтвердить формат enum и доступность генераторов значений в Exolynk.
- Выяснить, как в UI платформы включается режим редактирования и применить ограничения по ролям.
- Определить, поддерживает ли канбан переходы с проверкой суммы без кастомного кода.
