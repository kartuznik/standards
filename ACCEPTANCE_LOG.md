# Acceptance Log (cross-project)

Журнал продуктовых приёмок по `ACCEPTANCE_CHECKLIST.md`.
Нейтральные записи — в GitHub, детали с инфраструктурой — только локально.
Не дублировать в продуктовых репозиториях.

Формат записи: продукт / дата / git SHA / вердикт / покрытие чек-листа / метод / замечания / кто принял.

---

## Принятый риск runtime-доступа — 2026-11-12 — ACCEPTED

Ефим принял риск, связанный с порядком restart и live smoke после runtime-изменений, и связанные с ним ограничения на действия агента приёмочного контура. Это не продуктовый PASS. Инфраструктурные детали — только в локальной копии журнала.

Подпись: `Принятый риск runtime / 2026-11-12 / Ефим / ACCEPTED`

---

## Watchdog Bot — 2026-08-05 — PASS (demo-ready)

| Поле | Значение |
|---|---|
| Продукт | Watchdog Bot (`@watchdogassistant_bot`, `kartuznik/watchdog_bot`) |
| Дата | 5 августа 2026 |
| Git SHA | `c86e9eb` |
| Вердикт | **PASS — demo-ready / sale-ready** |
| Чек-лист | **13 / 13** секций `ACCEPTANCE_CHECKLIST.md` пройдены |
| Статус | **sale-freeze** с даты приёмки (см. `RULES.md` § sale-freeze) |
| Кто принял | agent self-audit + АПРУВ Архитектора |

### Метод приёмки

1. Живые Telegram-пробы трёх классов запросов (креатив без поиска; новостной с поиском «сегодня»; запрос без свежих данных / honesty).
2. Логи retrieval (query gate, relevance drops, `tavily_days` / `source_date_range`, freshness degradation, evidence_*).
3. Проверки Grafana (dashboard, contact point, alert rules) и admin-панели (login + `/api/usage`).

### Неблокирующие замечания

1. Отдельный out-of-process smoke **не крутит** Prometheus-метрики long-running процесса `bot` — для demo смотреть Grafana после обычных сообщений боту.
2. Живой fire Grafana-алерта в прогоне **не дергали** (избежание шума); доставка подтверждена конфигом (`$__env{…}`) и in-bot health-alert путём.
3. Для запросов с **именами собственными** внутри окна свежести возможны loosely-related посты при пересечении общих токенов (напр. «деревня»); кейс honesty закрыт отдельной пробой (комета Галлея / дата данных).

### Подпись

`Watchdog Bot / 2026-08-05 / agent self-audit / c86e9eb — PASS (demo-ready)`
