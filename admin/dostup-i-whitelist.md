# Доступ, SpringAuth и Denizen

Как выдаётся роль «игрок», что делает PHP на `springrp.ru/auth-bot/` и какие скрипты Denizen держат whitelist на Paper.

## Цепочка целиком

```text
Discord /auth → PHP (RCON или очередь) → LuckPerms «игрок» 30d → marallyzen.play → вход на сервер
Лаунчер → launcher.php (код) → Discord /code → offline-запуск с подтверждённым ником
```

| Компонент | Где | Назначение |
|-----------|-----|------------|
| SpringAuth | Discord Application | `/auth`, `/code`, `/help` |
| `webhook.php` | REG.RU | Interactions Endpoint Discord |
| `claim.php` | REG.RU | Очередь ников для Paper, если RCON с хостинга недоступен |
| `launcher.php` | REG.RU | Коды входа для лаунчера |
| `auth_bot_claim.dsc` | Paper | Забирает очередь и выполняет `lp user … addtemp игрок 30d` |
| `player_whitelist.dsc` | Paper | Kick без `marallyzen.play` |
| `luckperms_integration.dsc` | Paper | Группа «игрок» и право `marallyzen.play` |

Конфиг Denizen: `plugins/Denizen/data/auth_bot.yml` — URL `claim.php`, `poll.php` и общий `secret` (тот же, что `CLAIM_SECRET` в `.env` на сайте).

## `auth_bot_claim.dsc`

**Задача:** выдать LuckPerms роль, если PHP не смог достучаться до RCON и положил ник в `pending.json`.

**Триггер:** каждые **15 секунд** (`on delta time secondly every:15`).

**Шаги:**

1. Читает `data/auth_bot.yml`; без валидного `secret` — выход.
2. Флаг `marallyzen_auth_bot_busy` (30 с) — не дублировать запросы.
3. Опционально дергает `poll.php` (с Discord это заглушка `ok discord`).
4. `GET claim.php?secret=…` — получает список ников и **очищает** очередь на сайте.
5. Для каждого ника `^[A-Za-z0-9_]{3,16}$` выполняет:
   ```text
   lp user <ник> parent addtemp игрок 30d
   ```

Если RCON с REG.RU работает, `/auth` выдаёт роль сразу в PHP — Denizen просто не получает ников в очереди.

## `player_whitelist.dsc`

**Задача:** не пускать на сервер без права **`marallyzen.play`**.

**При входе (`on player logs in`):**

- OP и игроки с `marallyzen.play` — проходят.
- Остальные — kick: *«Напиши боту в Discord: /auth \<ник\>»*.

**Каждые 30 минут (`on system time minutely every:30`):**

- Обходит онлайн-игроков без `marallyzen.play` и кикает (истёк temp parent «игрок»).

Staff (builder и выше) получает `marallyzen.play` через `luckperms_integration.dsc` и whitelist не блокирует.

## Деплой и проверка

**Сайт (FTP `www/springrp.ru/auth-bot/`):** `bot_lib.php`, `webhook.php`, `register.php`, `poll.php`, `claim.php`, `launcher.php`, `.env`, `.htaccess`.

**Discord Developer Portal:**

- Interactions Endpoint URL: `https://springrp.ru/auth-bot/webhook.php`
- User Install включён для команд в ЛС
- Один раз: `register.php?secret=…` или `node bot.js` — публикация slash-команд

**Paper:** залить `.dsc`, обновить `auth_bot.yml`, **`/ex reload`**.

**Проверка:**

1. `/auth TestNick` в Discord → через ~15 с или сразу роль «игрок».
2. Заход без роли → kick с текстом про Discord.
3. Лаунчер: ник → код → `/code` → «Играть».
