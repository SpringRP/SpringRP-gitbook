# Telegram-авторизация (AuthTG)

На сервере стоит **AuthTG 3.0**: привязка Minecraft-ника к Telegram, опциональный пароль, 2FA, друзья и чат с ботом.

Официально плагин рассчитан на Paper 1.16–1.21 (`api-version: 1.19`). SpringRP на **26.1.2** — это новее. Если после рестарта плагин не загрузится, смотри `logs/latest.log`.

Velocity не используется: один Paper, в конфиге `velocity.enabled: false`.

Сервер в `online-mode=false`, поэтому классический `/register` + `/login` имеет смысл для защиты ников. Сейчас в конфиге стоит безопасный старт: **`notRegAndLogin: true`**, чтобы игроки не застряли до настройки бота.

## Что нужно от тебя

1. В Telegram открой [@BotFather](https://t.me/BotFather).
2. `/newbot` — имя вроде `SpringRP Auth`, username вроде `springrp_auth_bot`.
3. Скопируй токен и username (без `@`).
4. Пришли их в чат (или впиши сам в `plugins/AuthTG/config.yml`):

```yaml
tg: true
bot:
  token: "123456:ABC..."
  username: "springrp_auth_bot"
```

5. **Полный рестарт** сервера (не `/ex reload`). Бот поднимается только при старте. `/authtg reload` не подхватывает новый токен.

Если бот не отвечает (хостинг в РФ), в том же конфиге включи `bot.proxy`.

## Привязка аккаунта

Игрок онлайн в Minecraft, затем в боте:

```text
/start
```

или `/link`. Бот просит ник, при включённом пароле — пароль, затем код. В игре:

```text
/code <код>
```

Сообщение в боте, которое начинается с `#`, уходит в игровой чат от привязанного ника.

## Когда включать пароль

В `plugins/AuthTG/config.yml`:

```yaml
notRegAndLogin: false
authNecessarily: false
```

После рестарта новички делают `/register <пароль> <повтор>`, затем `/login <пароль>`. Уже играющие аккаунты тоже должны зарегистрироваться.

Обязательное подтверждение каждого входа через Telegram:

```yaml
notRegAndLogin: true
authNecessarily: true
authNecessarilyPrefer: "TG"
```

## Права LuckPerms

Owner: `authtg.admin`, reload, ban/kick/mute, unlink, spawn, broadcast. Admin: kick/mute/unmute.

Внутри бота отдельно выдаётся роль `/admin add <ник>` (игрок должен быть онлайн).

## Команды

```text
/authtg reload
/admin add <ник>
/code <код>
/unlink <ник>
/setspawn
/setspawn none
/friend
/tgbc <текст>
```

Полный список — в README плагина: https://github.com/Ezhik24/AuthTG
