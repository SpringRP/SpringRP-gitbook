# Доступ, SpringAuth и Denizen

Как выдаётся роль «игрок», что делает PHP на `springrp.ru/auth-bot/` и какие скрипты Denizen держат доступ на Paper.

## Цепочка целиком

```text
Лаунчер (Microsoft или ник) → попытка входа на сервер
→ player_bind.dsc (on player logs in): bound:1 + marallyzen.play? → мир
→ иначе bind.php gate → экран отключения (kick) с 8-символьным кодом
→ Discord /code → bound.json + очередь LP → rejoin через ~15 с
→ marallyzen.play → вход в мир
```

| Компонент | Где | Назначение |
|-----------|-----|------------|
| SpringAuth | Discord Application | `/code`, `/help` |
| `webhook.php` | REG.RU | Interactions Endpoint Discord |
| `bind.php` | REG.RU | Gate/issue/status для Denizen (header `X-Auth-Secret`) |
| `claim.php` | REG.RU | Очередь ников для Paper, если RCON с хостинга недоступен |
| `admin.php` | REG.RU | unbind, regrant, диагностика (ADMIN_SECRET) |
| `player_bind.dsc` | Paper | Login gate, kick screen с кодом (без freeze/чата) |
| `auth_bot_claim.dsc` | Paper | Забирает очередь и выполняет `lp user … addtemp игрок 30d` |
| `player_whitelist.dsc` | Paper | Kick без `marallyzen.play` каждые 30 мин (истёк срок) |
| `luckperms_integration.dsc` | Paper | Группа «игрок», `marallyzen.play`, default без gameplay |

Для проверки `marallyzen.play` до появления игрока в мире Denizen использует
Vault permissions API. На Paper 26.1.2 обязателен `VaultUnlocked 2.20.0` (или
другой совместимый Vault-провайдер) вместе с LuckPerms. После установки или
обновления VaultUnlocked нужен полный рестарт сервера, `/ex reload` недостаточно.

**Авторизован** = `bound:1` в `bound.json` **и** активное право `marallyzen.play` (temp parent «игрок»).

Конфиг Denizen: `plugins/Denizen/data/auth_bot.yml` — URL `claim.php`, `poll.php`, `bind.php`, `bot_name`, `secret` (CLAIM) и `bind_secret` (BIND).

## `player_bind.dsc` (v8)

**Задача:** не пускать неавторизованных в мир; код — только на **экране отключения**.

**При входе (`on player logs in`):**

1. Есть `marallyzen.play` → проверка `bind.php?action=status` → `bound:1` → пускать.
2. Есть `marallyzen.play`, но не `bound:1` → kick с новым кодом (re-auth).
3. Нет `marallyzen.play` → `bind.php?action=gate`:
   - `wait` → kick «роль выдаётся, зайдите через ~15 с»;
   - `code:XXXXXXXX` → kick с инструкцией и **8-символьным кодом** на экране.

Freeze, actionbar, narrate и цикл ожидания **убраны** — игрок не видит мир до привязки.

**При успешном join (`on player joins`):** одноразовое приветствие, если есть `marallyzen.play`.

## `auth_bot_claim.dsc`

**Задача:** выдать LuckPerms роль, если PHP не смог достучаться до RCON и положил ник в `pending.json`.

**Триггер:** каждые **15 секунд**. Запросы к `claim.php` / `poll.php` с header `X-Auth-Secret` (CLAIM_SECRET).

Если RCON с REG.RU работает, `/code` выдаёт роль сразу в PHP — Denizen просто не получает ников в очереди.

## `player_whitelist.dsc`

**Задача:** кикать игроков с истёкшим temp parent «игрок» (онлайн без `marallyzen.play`).

**Каждые 30 минут:** kick с текстом про повторный вход и код на экране отключения.

Join без права обрабатывает `player_bind.dsc` на этапе login — мгновенный kick screen, не чат.

## Настройка Discord (SpringAuth)

1. [Discord Developer Portal](https://discord.com/developers/applications) → приложение SpringAuth.
2. **Bot** → скопировать токен в `DISCORD_BOT_TOKEN`. Права при инвайте: **Manage Roles**, **Manage Nicknames**, **Send Messages**.
3. Пере-пригласить бота с правами **Manage Roles** + **Manage Nicknames** (не путать с Change Nickname):  
   `https://discord.com/api/oauth2/authorize?client_id=1543866009805656114&permissions=402653184&scope=bot%20applications.commands`
4. OAuth2 URL Generator → scope `bot` → пригласить на Discord-сервер проекта.
5. На сервере Discord:
   - Создать роль **@Игрок** (или использовать существующую).
   - Роль бота **SpringAuth** должна быть **выше** @Игрок и выше обычных участников — иначе ник и роль не выдаются.
   - У роли бота включены **Manage Roles** и **Manage Nicknames**.
   - **Владелец Discord-сервера:** бот **не может** сменить ваш ник — это ограничение Discord. Для теста используйте второй аккаунт или смените ник вручную.
   - @Игрок видит игровые каналы.
6. Скопировать **Guild ID** (ПКМ по серверу → Copy Server ID) → `DISCORD_GUILD_ID` в `.env`.
7. Скопировать **Role ID** роли @Игрок → `DISCORD_PLAYER_ROLE_ID` в `.env`.
8. Interactions Endpoint URL: `https://springrp.ru/auth-bot/webhook.php`
9. User Install включён для команд в ЛС.
10. Один раз: `register.php?secret=…` (CLAIM_SECRET) — публикация `/code`, `/help`.

Дополнительно в `.env`:

```env
DISCORD_GUILD_ID=
DISCORD_PLAYER_ROLE_ID=
DISCORD_BOT_INVITE_NAME=springauth
CLAIM_SECRET=
BIND_SECRET=
ADMIN_SECRET=
DEBUG_LOG=0
```

`DISCORD_BOT_INVITE_NAME` — имя бота в тексте kick screen (без `@`).

## Деплой и проверка

**Сайт (FTP `www/springrp.ru/auth-bot/`):** `bot_lib.php`, `webhook.php`, `webhook_worker.php`, `register.php`, `poll.php`, `claim.php`, `bind.php`, `admin.php`, `.htaccess`, `reserved_nicks.json`, `.env`.

**Paper (SFTP):** залить `player_bind.dsc`, `player_whitelist.dsc`, `auth_bot_claim.dsc`, `luckperms_integration.dsc`, обновить `auth_bot.yml`, **`/ex reload`**.

**Проверка:**

1. Лаунчер: ник без кодов → игра стартует.
2. Join без LP: мгновенный kick screen с 8-символьным кодом (мир не виден).
3. Discord `/code XXXXXXXX` → bound + LP в очереди.
4. Rejoin через ~15 с → вход в мир.
5. Rejoin сразу после `/code` → kick «роль выдаётся».
6. Истёк LP → kick screen с новым кодом.
7. 6+ неверных `/code` → rate limit в Discord.
