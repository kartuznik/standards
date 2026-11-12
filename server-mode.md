# server-mode.sh — режимы сервера

Скрипт: `/opt/bots/server-mode.sh`.

Скрипт переводит хост между четырьмя режимами. Он вызывает `systemctl start/stop/enable/disable` и `docker start/stop`. Тексты юнитов, `docker-compose.yml` и sysctl он не меняет.

Запуск от root. Если процесс запущен не от root, скрипт перезапускает себя с повышением прав.

## Команды

```bash
/opt/bots/server-mode.sh status
/opt/bots/server-mode.sh list
/opt/bots/server-mode.sh --dry-run production
/opt/bots/server-mode.sh production
/opt/bots/server-mode.sh sleep
/opt/bots/server-mode.sh demo
/opt/bots/server-mode.sh full
```

`--dry-run` можно поставить и после имени режима. Для `status` и `list` флаг ничего не меняет: эти команды и так только читают.

Коды выхода:

| Код | Когда |
|---|---|
| 0 | Успех. Отсутствующий юнит или контейнер сюда же: строка предупреждения, скрипт идёт дальше |
| 1 | Нет прав или неизвестная команда |
| 2 | Старт или стоп существующего юнита/контейнера не удался, имя из denylist попало в stop, либо в demo/full сработал тормоз по свободной памяти |

Повторный запуск того же режима догоняет расхождение: уже верное состояние пропускается.

## Режимы

База, включена во всех четырёх режимах (`enable`, если юнит ещё disabled, затем `start`, если он не active):

- `order-hunter-bot`
- `vacancy-hunter`
- `vacancy-hunter.timer`
- `shopbot`

Oneshot `vacancy-hunter-run.service` и `surveybot-check.service` скрипт не enable: секции `[Install]` у них нет, их запускает таймер.

| Режим | Дополнительно включено | Выключено (stop, затем disable) | Docker |
|---|---|---|---|
| production | — | `shopbot-web`, `surveybot`, `surveybot-web`, `surveybot-check.timer`, `kuzya-bot`, `botcore`, `botcore-api` | все 4 контейнера watchdog остановлены |
| sleep | `botcore`, `botcore-api` | тот же набор, что у production | все 4 контейнера watchdog остановлены |
| demo | sleep + `shopbot-web`, `surveybot`, `surveybot-web`, `kuzya-bot`, `surveybot-check.timer` | в управляемом наборе пусто | redis → worker → bot → admin-panel |
| full | тот же состав, что demo | тот же состав, что demo | тот же порядок |

`demo` и `full` — одна маска процессов. Живое состояние важнее файла режима: если процессы не совпали с маской, заголовок `custom`.

`status` считает сервис включённым, когда он одновременно `active` и `enabled`. Выключенным — когда он не `active`. Поэтому `shopbot` в состоянии active, но disabled, не считается режимом production, пока скрипт его явно не enable.

Сценарии:

- production — обычная работа. Хантеры и приём платежей. Админка ShopBot и остальные боты выключены. Таймер `surveybot-check.timer` выключен (disable + stop).
- sleep — то же плюс лёгкая пара `botcore` / `botcore-api` для проверки, что хост жив.
- demo — показ партнёру или скринкаст. Надолго не оставлять.
- full — тот же состав для разработки и ручной проверки.
