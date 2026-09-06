# INFRA_ACCESS — доступ к боевому серверу

Кросс-проектная памятка для Архитектора и агента. Секреты, приватные ключи и голые адреса сюда не пишем.

---

## SSH и scp — только алиас `aeza`

SSH и scp к боевому серверу работают **только** через алиас `aeza` из пользовательского `~/.ssh/config` на машине Архитектора.

Ожидаемые поля в `.ssh/config` (значения хоста/ключа на машине Архитектора, не в этом файле):

```
Host aeza
    Port 2222
    User root
```

Авторизация — **ключом** (не паролем).

Любые команды для Архитектора формулировать **строго с алиасом** `aeza`. Не подставлять IP, не указывать `-p 22`, не использовать `root@<ip>`.

Правильный пример скачивания свежего бэкапа:

```bash
scp aeza:/opt/backups/latest.tar.gz .
```

Windows (PowerShell), тот же алиас:

```powershell
scp aeza:/opt/backups/latest.tar.gz $env:USERPROFILE\Downloads\backup_latest.tar.gz
```

### Запрет: голый IP и порт 22

**Запрещено** подключаться по голому IP на дефолтном порту 22. Порт 22 закрыт; такая команда даёт `Connection timed out` и выглядит как «сервер мёртв», хотя SSH жив на `aeza` / порту 2222.

Нельзя (антипример):

```bash
scp root@<IP>:/opt/backups/latest.tar.gz .
ssh -p 22 root@<IP>
```

---

## Бэкапы

| Что | Где / как |
|---|---|
| Каталог | `/opt/backups` (права каталога `700`) |
| Свежий архив | символическая ссылка `/opt/backups/latest.tar.gz` |
| Имя снимка | `backup_YYYYMMDD_HHMMSS.tar.gz` |
| Ротация | архивы старше **7 дней** удаляются скриптом |
| Timer | `backup-bot.timer` — ежедневно **03:00 UTC**, unit `backup-bot.service` |

Скрипт: `/opt/bots/backup/create_backup.sh`. Ручной запуск (на сервере): `systemctl start backup-bot.service`. Статус таймера: `systemctl status backup-bot.timer`.
