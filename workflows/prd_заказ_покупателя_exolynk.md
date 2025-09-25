# ТЗ/PRD: «Заказ покупателя» — UI + логика на платформе Exolynk

## 0) Контекст и цель
Собрать MVP-процесс «Заказ покупателя» на **Exolynk** (low‑code на PostgreSQL) с разграничением ролей (**Sales**, **Manager**, **Logistics**, **System**), единым канбаном, карточкой заказа в светлой теме и автоматизациями.

Задача — показать, что платформа позволяет:
- разделить управление **состояниями** по ролям (статусы вычисляет система);
- защитить изменения (режим «Изменить» + запрос на сохранение);
- слать уведомления и запускать сервисные проверки.

**Документация Exolynk:**
- Getting started: https://exolynk.app/docs/docs/usage/getting-started/
- Rune API: https://exolynk.app/docs/docs/api/rune/
- REST API: https://exolynk.app/docs/docs/api/rest/
- Templates: https://exolynk.com/en/templates/

---

## 1) Термины
- **Статус заказа** (жизненный цикл, считает система): **Открыт → Исполняется → Закрыт**.
- **Состояние (канбан‑колонка)**: **Новый → На утверждение → Комплектация → Готово к отгрузке → Заказ закрыт**.  
  Финальная колонка одна: **Заказ закрыт**. На карточке в ней показываем бейдж: 🟦 **Выполнен** (успех) или 🔴 **Закрыт неуспешно** (неуспех).
- **Логистика (`shipment_state`)**: **Комплектация → Готово к отгрузке → Отгружено** (служебные атрибуты карточки, участвуют в логике, но не отдельные канбан‑колонки).
- **Оплата (`payment_state`)**: **Нет / Да**.

---

## 2) Роли и права (управляют только Состояниями)
| Роль | Разрешённые переходы |
|---|---|
| **Sales** | **Новый → Комплектация** (если сумма ≤ 10 000) • **Новый → На утверждение** (если сумма > 10 000) • (опц.) «Заказ закрыт» · *неуспешно* |
| **Manager** | **На утверждение → Комплектация** (кнопка **«Разрешить комплектацию»**) • подтверждает оплату на шаге «Готово к отгрузке» |
| **Logistics** | **Комплектация → Готово к отгрузке** |
| **System** | Ставит **Отгружено** после подтверждения оплаты • Переводит в **«Заказ закрыт (успешно)»** при `shipment=Отгружено` и `payment=Да` • Вычисляет Статус: Открыт/Исполняется/Закрыт |

Все видят **один и тот же канбан**, но двигать могут только «свои» шаги (кнопки вне прав — неактивны).

---

## 3) Данные (модели и поля)

### 3.1 Model: `order`
- `id` (uuid)
- `number` (text, автонумератор)
- `client_id` (uuid, FK → `client`)
- `created_at` (timestamptz, дефолт NOW)
- `status` (enum: `OPEN|EXECUTION|CLOSED`) — **считает система**
- `state` (enum: `NEW|APPROVAL|PICKING|READY|CLOSED`) — **канбан**
- `shipment_state` (enum: `PICKING|READY|SHIPPED|null`)
- `payment_state` (enum: `NO|YES`)
- `amount_total` (numeric(14,2))
- `closed_success` (bool|null) — `true` = выполнен, `false` = закрыт неуспешно
- `closed_reason` (text|null)
- `ship_date` (date|null) — назначает Logistics
- `comment` (text|null)
- `created_by` (uuid)

### 3.2 Model: `order_item`
- `id` (uuid)
- `order_id` (uuid, FK)
- `sku` (text, обязателен)
- `name` (text)
- `qty` (numeric(14,3))
- `price` (numeric(14,2))
- `amount` (generated always `qty*price`)

### 3.3 Model: `client`
- базовые поля (имя, контакты)

**Именование полей** — `snake_case`. Значения enum — CAPS, как указано.

---

## 4) Представления и интерфейсы

### 4.1 View: `orders_kanban`
- Колонки: **Новый | На утверждение | Комплектация | Готово к отгрузке | Заказ закрыт**.  
- Карточка в финальной колонке имеет бейдж: 🟦 **Выполнен** (успех) / 🔴 **Неуспешно** (провал).
- Фильтры: «Мои», «Все», быстрые по состояниям и по статусу (Открыт/Исполняется/Закрыт).

### 4.2 User Layout: `order_detail`
Светлая тема, компактная карточка:
- шапка: Заголовок + мета `№` и `Создан`; кнопки: **Изменить** / **Комплектация**;
- **прогресс‑бар состояний** (один блок: Новый → На утверждение → Комплектация → Готово к отгрузке → Заказ закрыт);
- компактные поля в линию: **Клиент**, **Дата отгрузки (логистика)** [readonly], **Оплата** [readonly];
- таблица позиций (внутренний скролл): **Артикул | Наименование | Кол‑во | Цена | Сумма | ×**;  
  — добавление строки — только в режиме **Изменить** (кнопка «+» внизу списка);
- нижняя единая плашка на 2 равных блока: **Комментарий** (textarea с плейсхолдером) слева, **Суммы** справа (Сумма товаров, Итого);
- **Защита от случайных изменений**: редактирование только после **«Изменить»**; при уходе с несохранёнными правками — «Сохранить изменения?».

### 4.3 Видимость кнопок по ролям (в карточке)
- **Sales**: «Изменить», «Комплектация», (опц.) «Закрыть неуспешно».
- **Manager**: «Разрешить комплектацию» (на шаге «На утверждение»), «Подтвердить оплату» (на «Готово к отгрузке»).
- **Logistics**: «Готово к отгрузке».
- Недоступные действия — `disabled` с подсказкой «Недоступно в вашей роли/на этом шаге».

---

## 5) Правила переходов и вычисления Статуса

### 5.1 Триггеры
- **Sales → «Комплектация» (из «Новый»):**  
  `amount_total ≤ 10000` → `state=PICKING` (Комплектация), `status=EXECUTION`, уведомление в Logistics.  
  `amount_total > 10000` → `state=APPROVAL` (На утверждение), `status=EXECUTION`, уведомление в Manager.
- **Manager → «Разрешить комплектацию» (из «На утверждение»):**  
  `state=PICKING`, задача Logistics.
- **Logistics → «Готово к отгрузке»:**  
  `shipment_state=READY`, `ship_date` (required), уведомление в Manager **«Подтвердить оплату»**.
- **Manager → «Подтвердить оплату»:**  
  `payment_state=YES` → **System** ставит `shipment_state=SHIPPED` и переводит `state=CLOSED` с `closed_success=true`, `status=CLOSED`.
- **(Опц.) Sales → «Закрыть неуспешно»:**  
  `state=CLOSED`, `closed_success=false`, `status=CLOSED`, требуется `closed_reason`.

### 5.2 Расчёт `status`
1) `state=CLOSED` → `status=CLOSED`.  
2) Иначе `state ∈ {APPROVAL,PICKING,READY}` → `status=EXECUTION`.  
3) Иначе → `status=OPEN`.

---

## 6) Автоматизации (Workflows / Rune / Services)

### 6.1 Workflows
- `wf_sales_to_picking`: on action **sales.to_picking** → ветвление по сумме (10 000).  
- `wf_manager_allow_picking`: on action **manager.allow_picking**.  
- `wf_logistics_ready`: on action **logistics.ready**.  
- `wf_manager_confirm_payment`: on action **manager.confirm_payment** → call service `svc_close_success`.

### 6.2 Services / Templates (уведомления)
- `svc_notify_manager_approval(order_id)` → шаблон `tpl_order_approval_request` (email/webhook).  
- `svc_notify_logistics_picking(order_id)` → `tpl_order_to_picking`.  
- `svc_notify_manager_payment(order_id)` → `tpl_payment_confirmation_request`.  
- `svc_close_success(order_id)` — устанавливает `shipment_state=SHIPPED`, `state=CLOSED`, `closed_success=true`, `status=CLOSED`.

> Уведомления — в правую панель «Сообщения» и/или email/webhook.

---

## 7) Права доступа (Roles & ACL)
- **Sales**: R/W свои заказы; действия: `sales.to_picking`, `sales.close_fail`.  
- **Manager**: R/W все; действия: `manager.allow_picking`, `manager.confirm_payment`.  
- **Logistics**: R/W логистические поля; действие: `logistics.ready`. Нет создания заказов, нет установки `PICKING`.  
- **System**: сервисные действия/скрипты только через Workflow.

---

## 8) REST / Rune интеграции (схема)
- REST авто‑эндпоинты моделей `order`, `order_item`, `client` (CRUD, фильтры).  
- Action‑эндпоинты (через Rune/Workflow), имена действий:
  - `POST /api/order/{id}/actions/sales.to_picking`
  - `POST /api/order/{id}/actions/manager.allow_picking`
  - `POST /api/order/{id}/actions/logistics.ready`
  - `POST /api/order/{id}/actions/manager.confirm_payment`

Все действия валидируют состояние и роль; при ошибке — 409 + код причины.

---

## 9) UX‑детали карточки (светлая тема)
- плоские тонкие рамки, компактные поля;
- «Создан: …» — малым шрифтом в шапке;
- таблица позиций со скроллом (5–10 строк по высоте), колонка «×» — узкая;
- добавление строки — только в режиме **Изменить** («+ Добавить строку» внизу списка);
- кнопка **«Комплектация»** активна только в `state=NEW` и в режиме **Изменить**;
- «Сохранить изменения?» — **только если** есть правки и активен режим редактирования.

---

## 10) Именование артефактов (для внедрения)
- **Models**: `order`, `order_item`, `client`
- **Variables**: как в разделе 3 (snake_case)
- **Views**: `orders_kanban`, `orders_list`, `clients_list`
- **User Layouts**: `order_detail`, `client_detail`
- **Workflows**: `wf_sales_to_picking`, `wf_manager_allow_picking`, `wf_logistics_ready`, `wf_manager_confirm_payment`
- **Services**: `svc_notify_manager_approval`, `svc_notify_logistics_picking`, `svc_notify_manager_payment`, `svc_close_success`
- **Templates**: `tpl_order_approval_request`, `tpl_order_to_picking`, `tpl_payment_confirmation_request`
- **Actions (UI‑кнопки)**: `sales.to_picking` («Комплектация»), `manager.allow_picking` («Разрешить комплектацию»), `logistics.ready` («Готово к отгрузке»), `manager.confirm_payment` («Подтвердить оплату»), `sales.close_fail` («Закрыть неуспешно»).

---

## 11) Критерии приёмки (MVP)
1) Все роли видят один канбан; недоступные действия — неактивны.  
2) Sales из «Новый» переводит:  
   — ≤10 000 → «Комплектация», `status=EXECUTION`, уведомление Logistics;  
   — >10 000 → «На утверждение», `status=EXECUTION`, уведомление Manager.  
3) Manager из «На утверждение» переводит в «Комплектация».  
4) Logistics переводит «Комплектация → Готово к отгрузке» и задаёт дату.  
5) Manager подтверждает оплату → System ставит «Отгружено» и переводит карточку в «Заказ закрыт» с бейджем **Выполнен** (`status=CLOSED`).  
6) Альтернатива: «Закрыть неуспешно» запрашивает причину и уводит в «Заказ закрыт» с бейджем **Неуспешно** (`status=CLOSED`).  
7) Любые изменения возможны только после «Изменить»; при несохранённых правках — запрос «Сохранить изменения?».  

---

## 12) Вне объёма (сейчас)
- Реальные платёжные интеграции (оплата подтверждается вручную Manager).  
- Полноценная мобильная вёрстка (опционально).  
- Поиск/каталог товаров — простой ввод SKU/Name.

