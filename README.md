# Exolynk Order MVP

Учебный кейс по запуску MVP процесса «Заказ покупателя» в low-code платформе **Exolynk**. Репозиторий служит примером того, как вести документацию, прототип и модель данных в едином пространстве.

## Структура
- `docs/overview.md` — точка входа, краткое описание проекта.
- `docs/requirements/` — бизнес-требования и PRD (`business_requirements.md`).
- `docs/process/` — детальные описания процессов (`process_sales_order.md`).
- `docs/tech/` — платформа, инструменты, интеграции (`tech_stack.md`).
- `docs/methodology/` — методологические материалы (SPS-TP).
- `docs/worklog/` — бэклог (`todo.md`) и журнал действий (`process_tool_log.md`).
- `diagrams/` — ERD и процессные диаграммы (`dbdiagram/`, `plantuml/`).
- `ui/` — прототип рабочего места продаж (`exolynk_sales_workspace.html`).
- `scripts/`, `data/` — вспомогательные материалы (при необходимости).

## Документы первой необходимости
| Файл | Назначение |
| --- | --- |
| `docs/overview.md` | Общее видение и ссылки на остальные разделы. |
| `docs/roadmap.md` | **Консолидированный роадмап и план задач проекта.** |
| `docs/requirements/business_requirements.md` | Формализованный набор требований (MVP). |
| `docs/process/process_sales_order.md` | Пошаговый сценарий процесса. |
| `docs/tech/tech_stack.md` | Платформа Exolynk, инструменты, вопросы интеграции. |
| `docs/methodology/SPS-TP.md` | Авторская модель информационных потоков. |
| `docs/worklog/todo.md` | Оперативный список задач. |
| `docs/worklog/process_tool_log.md` | Фактический лог действий. |
| `diagrams/dbdiagram/customer_order_erd.txt` | Актуальная ERD. |

## Прототип UI
1. Открыть `ui/exolynk_sales_workspace.html` в браузере.
2. Переключаться между вкладками «Заказы» и «Клиенты», чтобы проверить флоу.
3. Изменения состояния/позиций пока моковые (см. консоль).

## Процесс работы
1. Согласовываем изменения в `docs/requirements/` и `docs/process/`.
2. Фиксируем решения и вопросы в `docs/worklog/`.
3. Обновляем схемы в `diagrams/` и прототип в `ui/`.
4. После уточнения — готовим материалы для импорта в Exolynk (см. `tech_stack.md`).

## Следующие шаги (обновляется)
- Реализовать таблицы `Order`/`OrderItem` в окружении Exolynk.
- Добавить процессные диаграммы (PlantUML).
- Подготовить англоязычную версию ключевых документов.
