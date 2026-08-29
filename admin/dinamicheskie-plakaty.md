# Динамические плакаты (администратору)

Динамический плакат (`customposter`) использует настенный `paperposter1`. На стене остаётся чистая бумага из ресурспака — без чернил, голов и декора.

Текст появляется **только на вылете**. Denizen поднимает тот же лист как `paperposterfull`. SpringPosters по каналу `springrp:poster` отдаёт клиенту содержимое (шаблон, title/body/author, головы, декор). Fabric-мод `spring-poster-client` рисует ванильным шрифтом PAPER и головами **на той же геометрии** ItemDisplay. Лист, чернила и головы качаются и переворачиваются вместе. Отдельных TextDisplay и голов-пассажиров нет.

Без мода лаунчера на вылете видна только чистая бумага. Мод обязательный: он лежит в `mods/` внутри `fabric-26.1.2.zip` на `https://springrp.ru/launcher/game`. Play подхватывает его сам. Ванильный клиент текст не увидит.

PHP по-прежнему запекает PNG для предпросмотра в GUI и детерминированного кэша (`marallyzen-v1`). Сетка как у Marallyzen: заголовок жирный по центру, тело по центру, подпись `— автор` справа; цвета PAPER — `#1A1A1A` и `#555555`. Шаблоны `faces` и `wanted` используют раскладку одной/трёх голов из старого Marallyzen. Превью в GUI — этот же PNG (шрифт 5×7). В мире на вылете — ванильный шрифт, как в Marallyzen.

## Установка

1. Собрать `spring-posters` (`gradlew build`) — JAR попадает в `jar/`.
2. Загрузить `jar/SpringPosters-1.0.0.jar` в `/plugins/` по SFTP.
3. Загрузить `auth-bot/posters-api/` на REG.RU в `www/springrp.ru/posters-api/`.
4. На сайте создать `posters-api/.env` из `.env.example` с `SPRINGRP_POSTER_API_SECRET`.
5. На игровом сервере задать ту же переменную (или `api.secret` в `plugins/SpringPosters/config.yml`).
6. Собрать `spring-poster-client` (`gradlew jar`) и положить JAR в `mods/` внутри `fabric-26.1.2.zip`. Обновить `manifest.json` (`version`, `game.sha256`, `size`).
7. **Полный рестарт** Paper. `/reload` и PlugMan нельзя.
8. После старта: `/ex reload` не обязателен, если `posters.dsc` уже на диске до рестарта.
9. Игроки нажимают Play в лаунчере — скачается новый zip.

Проверка API без секрета должна отвечать `401`. С секретом одинаковый JSON даёт одинаковый `content_hash`.

## Команды

| Команда | Кто | Что делает |
|---|---|---|
| `/poster create` | `marallyzen.poster.create` | Ставит `customposter` на вертикальную грань |
| `/poster edit` | владелец / `edit.any` | Открывает GUI, глядя на плакат |
| `/poster clone` | admin | Два раза: источник, затем цель |
| `/poster owner <ник>` | admin | Сменить владельца |
| `/poster rerender` | admin | Повторно запечь PNG предпросмотра из БД / кэша |
| `/poster admin unlock` | admin | Снять edit lock |
| `/poster migrate` | admin | Запись БД для уже стоящего `customposter` |
| `/poster force` | admin | Снять клиентские сессии и вернуть чистые плакаты на стены |
| `/poster remove all` | admin | Сущности + Map bindings + строки БД |

Старые `/poster spawn|give|variant|rebuild` работают как раньше.

Shift+ПКМ по плакату открывает редактор. В текстовом диалоге отдельно задаются заголовок, основной текст и подпись автора. Обычный ПКМ — просмотр и анимация.

## Права

Группа «игрок»: `create`, `edit`, `edit.own`, `delete.own`.  
Builder: `edit.any`, `delete.any`.  
`marallyzen.poster.admin` покрывает всё.

## Если renderer недоступен

Активная версия не меняется. Черновик в GUI не становится активным. Игрок видит ошибку и может нажать «Сохранить» снова. Сервер не блокируется: HTTP идёт асинхронно. На вылете клиент всё равно рисует текст из сохранённых полей, даже если PNG предпросмотра не обновился.

## Данные

- SQLite: `plugins/SpringPosters/posters.db`
- Кэш PNG/пикселей: `plugins/SpringPosters/cache/<sha256>`
- Audit: `plugins/SpringPosters/audit.log`
- Канал клиента: `springrp:poster` (`session-start` bind / `session-end` unbind)

Не храните PNG в Denizen flags. Секреты, `.env` и `posters.db` в Git не коммитить.
