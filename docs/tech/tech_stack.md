# Технологии и интеграция

## Платформа Exolynk
- Используем Modeler (таблицы), Workflow, UI builder, Endpoints.
- MVP разворачиваем в тестовом окружении с преднастроенными таблицами `Kontakt`, `Firma`, `Bahnhof`, `Benutzer`, `Rolle`, `Rechte`.
- Новые сущности: `Order`, `OrderItem` (см. ERD).
- Журнал изменений состояний планируем вести через стандартный аудит Exolynk (дополнительная таблица не нужна).

## Модель данных (фрагмент)
- `Order`: идентификатор, номер, ссылка на клиента (`Kontakt`), суммы, состояния (`state`, `status`, `shipment_state`, `payment_state`), флаги закрытия.
- `OrderItem`: позиции с количеством и ценой.
- Enums: `state`, `status`, `shipment_state`, `payment_state`, `close_reason`.
- Типы полей: `Text`, `Number`, `Boolean`, `Date/DateTime`, `Enum (Selection)`, `UUID/Ident`, `Reference`.
- Полный ERD: `diagrams/dbdiagram/customer_order_erd.txt`.

## Инструменты
| Инструмент | Назначение | Файл/папка |
| --- | --- | --- |
| dbdiagram.io | Проектирование ERD, экспорт схем | `diagrams/dbdiagram/` |
| PlantUML | Диаграммы процессов, workflow | `diagrams/plantuml/` |
| VS Code + Git | Ведение репозитория, прототипирование | `.vscode/settings.json`, commit history |
| ChatGPT (ассистент) | Работа с документацией, кодом, логом | Настройка автозапуска в `.vscode/settings.json` |

## Методология SPS-TP
- SPS-TP (Signal → Process → Store → Transfer → Present) используется как каркас для построения информационного потока.
- Подробности: `docs/methodology/SPS-TP.md`.

## Интеграционные моменты
- Уведомления (`svc_*`) планируются как сервисы Exolynk (Rune/Endpoints) — пока описаны концептуально.
- REST и внешние интеграции не реализуются в MVP, но описаны в бэклоге (`docs/worklog/todo.md`).
- Требуется уточнить у платформенной команды: 
  - примеры реализации аудита состояний;
  - структуру кастомных уведомлений/вебхуков.

## Пробелы в документации платформы
- Локализация / переводы сущностей (описаний) — не полностью задокументировано.
- Инструкции по настройке UI workspace — есть общие статьи, но не хватает детальных примеров (зафиксировано в README как TODO). 
- План: поддерживать список вопросов и обновлять при получении ответов от команды Exolynk.
