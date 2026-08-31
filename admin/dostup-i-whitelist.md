# Доступ, SpringAuth и Denizen

Как выдаётся роль «игрок», что делает PHP на `springrp.ru/auth-bot/` и какие скрипты Denizen держат доступ на Paper.

## Цепочка целиком

```text
Лаунчер (Microsoft или ник) → вход на сервер
→ player_bind.dsc: код на экране, freeze 2m
→ Discord /code → bind.php + RCON → LuckPerms «игрок» 30d + роль @Игрок
→ marallyzen.play → unfreeze
```

| Компонент | Где | Назначение |
|-----------|-----|------------|
| SpringAuth | Discord Application | `/code`, `/help` |
| `webhook.php` | REG.RU | Interactions Endpoint Discord |
| `bind.php` | REG.RU | Выдача кодов для Denizen, проверка bound |
| `claim.php` | REG.RU | Очередь ников для Paper, если RCON с хостинга недоступен |
| `player_bind.dsc` | Paper | Freeze, код на экране, kick через 2m, unfreeze после LP |
| `auth_bot_claim.dsc` | Paper | Забирает очередь и выполняет `lp user … addtemp игрок 30d` |
| `player_whitelist.dsc` | Paper | Kick без `marallyzen.play` каждые 30 мин (истёк срок) |
| `luckperms_integration.dsc` | Paper | Группа «игрок» и право `marallyzen.play` |

Конфиг Denizen: `plugins/Denizen/data/auth_bot.yml` — URL `claim.php`, `poll.php`, `bind.php`, `bot_name` и общий `secret` (тот же, что `CLAIM_SECRET` в `.env` на сайте).

## `player_bind.dsc`

**Задача:** привязать Discord тем, у кого ещё нет `marallyzen.play`.

**При входе (`on player joins`):**

- OP и игроки с `marallyzen.play` — проходят без bind-flow.
- Остальные: `GET bind.php?action=issue&nick=…`, freeze (`is_immune` + отмена движения), actionbar с кодом каждые ~3 с.
- Цикл опрашивает `marallyzen.play` — после `/code` снимает freeze.
- Через **2 минуты** без привязки — kick с текстом про Discord.

**При выходе:** снимает флаги и immunity.

## `auth_bot_claim.dsc`

**Задача:** выдать LuckPerms роль, если PHP не смог достучаться до RCON и положил ник в `pending.json`.

**Триггер:** каждые **15 секунд**.

Если RCON с REG.RU работает, `/code` выдаёт роль сразу в PHP — Denizen просто не получает ников в очереди.

## `player_whitelist.dsc`

**Задача:** кикать игроков с истёкшим temp parent «игрок».

**Каждые 30 минут:** обходит онлайн без `marallyzen.play` (кроме тех, кто в bind-flow) и кикает.

Join без права обрабатывает `player_bind.dsc`, а не мгновенный kick.

## Настройка Discord (SpringAuth)

1. [Discord Developer Portal](https://discord.com/developers/applications) → приложение SpringAuth.
2. **Bot** → скопировать токен в `DISCORD_BOT_TOKEN`. Права при инвайте: **Manage Roles**, **Manage Nicknames**, **Send Messages**.
3. OAuth2 URL Generator → scope `bot` → пригласить на Discord-сервер проекта.
4. На сервере Discord:
   - Создать роль **@Игрок** (или использовать существующую).
   - Роль бота должна быть **выше** @Игрок в иерархии.
   - @Игрок видит игровые каналы.
5. Скопировать **Guild ID** (ПКМ по серверу → Copy Server ID) → `DISCORD_GUILD_ID` в `.env`.
6. Скопировать **Role ID** роли @Игрок → `DISCORD_PLAYER_ROLE_ID` в `.env`.
7. Interactions Endpoint URL: `https://springrp.ru/auth-bot/webhook.php`
8. User Install включён для команд в ЛС.
9. Один раз: `register.php?secret=…` или `node bot.js` — публикация `/code`, `/help`.

Дополнительно в `.env`:

```env
DISCORD_GUILD_ID=
DISCORD_PLAYER_ROLE_ID=
DISCORD_BOT_INVITE_NAME=springauth
```

`DISCORD_BOT_INVITE_NAME` — имя бота в тексте actionbar на сервере (без `@`).

## Деплой и проверка

**Сайт (FTP `www/springrp.ru/auth-bot/`):** `bot_lib.php`, `webhook.php`, `register.php`, `poll.php`, `claim.php`, `bind.php`, `.env`, `.htaccess`.

**Paper:** залить `player_bind.dsc`, `player_whitelist.dsc`, обновить `auth_bot.yml`, **`/ex reload`**.

**Проверка:**

1. Лаунчер: ник без кодов → игра стартует.
2. Join: freeze + actionbar с кодом.
3. Discord `/code` → LP + роль @Игрок + unfreeze.
4. 2 мин без кода → kick.
5. Повторный join привязанного с активной ролью → сразу игра.
