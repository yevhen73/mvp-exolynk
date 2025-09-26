# Data Model Alignment Notes

## Base Tables Already in Environment
- **Kontakt**: идентификаторы, ФИО, email, телефоны, `status`, `register_date`, привязка к `Firma`.
- **Firma**: справочник юридических лиц; на MVP не используем (связь остаётся необязательной).
- **Bahnhof**: сервисный каталог узлов/локаций; без изменений.
- **Benutzer / Rolle / Rechte**: стандартный RBAC, изменений не требуется.
- **Datei**: прикрепления документов — пока исключаем из MVP.

## Поля и типы Exolynk
Минимальный набор используемых типов согласно [документации](https://docs.exolynk.com/exolynk-modeler/variables/list):
- `Text` для строк (названия, комментарии, идентификаторы человека).
- `Number` для сумм, количеств.
- `Boolean` для флагов (`closed_success`).
- `Date` / `DateTime` для `ship_date`, `register_date`, отметок времени.
- `Enum` (Selection) для `order_state`, `order_status`, `payment_state`, `close_reason`.
- `UUID`/`Ident` для первичных ключей (`Order.id`, `Contact.id`).
- `Reference` для связей (`Order.customer_id → Kontakt.id`, `OrderItem.order_id → Order.id`).

Структуры/tuples, dynamic types и `Datei`/`Struct` не используем в MVP.

## Модель заказов (новые таблицы)
- **Order**
  - Поля: `id` (UUID), `order_no` (Text), `customer_id` (Reference → Kontakt), `status` (Enum OPEN/CLOSED), `state` (Enum NEW→APPROVAL→PICKING→READY→CLOSED), `payment_state` (Enum NO/YES), `shipment_state` (Enum PICKING/READY/SHIPPED), `ship_date` (Date), `amount_total` (Number), `closed_success` (Boolean), `closed_reason` (Enum), `comment` (Text), `created_at`, `updated_at` (DateTime), `created_by` (Reference → Benutzer).
- **OrderItem**
  - Поля: `id`, `order_id` (Reference → Order.id), `sku` (Text), `name` (Text), `qty` (Number), `price` (Number), `amount` (Number).
- **Product** (опционально, как справочник SKU; можно отложить, пока заполняем позиции напрямую).
- **OrderStateHistory** — отказываемся, опираемся на системный аудит Exolynk.

## Вопросы/решения
- Связь `Order.customer_id` на `Kontakt.id` подтверждена; `Firma` не обрабатываем в MVP.
- Историю состояний будем брать из системного лога Exolynk — отдельная таблица не нужна.
- Прикрепления (`Datei`) и связанные сервисы — за рамками MVP.

## To-Do
- Подготовить финальный ERD для импорта (формат dbdiagram / Exolynk Modeler).
- Уточнить у платформенной команды типы перечислений (Enum) и названия Selection-значений.
- Создать схемы `Order`/`OrderItem` в тестовом окружении.
