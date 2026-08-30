# Плакаты (администратору)

Все виды плакатов (`poster1`–`poster10`, `oldposter`, `paperposter*`, `customposter`) редактируются одним GUI. На стене остаётся чистая бумага из ресурспака — без чернил, голов и декора.

Текст появляется **только на вылете**. Denizen поднимает лист полной модели (`posterfull` / `paperposterfull` / `oldposterfull_*`). SpringPosters по каналу `springrp:poster` отдаёт содержимое и тип плаката. Fabric-мод `spring-poster-client` рисует ванильным шрифтом на той же геометрии ItemDisplay, но **сетка чернил следует полю бумаги в текстуре**, как в Marallyzen:

| Семья | Текстура | Поле 110×146 | Сдвиг |
|---|---|---|---|
| `poster1`–`10` | `posterfull` | поле 12 / 86×122, `#2B1B0E`, заголовок CAPS | 0 |
| `paperposter*` / `customposter` | `paperposterfull` | поле 14 / 82×116, `#1A1A1A` / автор `#555555` справа | +0.095 (бумага в UV 0.20–0.99) |
| `oldposter` | `oldposterfull_*` | поле 14 / 82×108, тот же коричневый, без CAPS | 0 (рваный низ) |

Лист, чернила и головы качаются и переворачиваются вместе. Отдельных TextDisplay и голов-пассажиров нет.

Без мода лаунчера на вылете видна только чистая бумага. Мод обязательный: он лежит в `mods/` внутри `fabric-26.1.2.zip` на `https://springrp.ru/launcher/game`. Play подхватывает его сам. Ванильный клиент текст не увидит.

PHP по-прежнему запекает PNG для предпросмотра в GUI и детерминированного кэша (`marallyzen-v1`). Сетка как у Marallyzen: заголовок жирный по центру, тело по центру, подпись `— автор` справа; цвета PAPER — `#1A1A1A` и `#555555`. Шаблоны `faces` и `wanted` используют раскладку одной/трёх голов из старого Marallyzen. Превью в GUI — этот же PNG (шрифт 5×7). В мире на вылете — ванильный шрифт, как в Marallyzen.

## Владение

Флаг Denizen `custom` и колонка `player_owned` — одно и то же.

| Кто ставит | Как | Владение |
|---|---|---|
| Игрок | крафт + ПКМ, `/poster create` | свой (`custom=true`) |
| Админ / builder | предмет или `/poster spawn` | административный (`custom=false`) |

Свой плакат редактирует владелец (`edit` / `edit.own`). Чужой свой — только `edit.any` или `poster.admin`. Административный плакат редактирует только `marallyzen.poster.admin` или OP. Одного `edit.any` недостаточно.

Игрок не может сломать блок-опору административного плаката. Взрыв вырезает эту опору из списка блоков, поршень и огонь отменяются. Чтобы снести такую опору, нужны OP, `poster.admin` или `poster.break.admin`.

## Установка

1. Собрать `spring-posters` (`gradlew build`) — JAR попадает в `jar/`.
2. Загрузить `jar/SpringPosters-1.0.0.jar` в `/plugins/` по SFTP.
3. Загрузить `auth-bot/posters-api/` на REG.RU в `www/springrp.ru/posters-api/`.
4. На сайте создать `posters-api/.env` из `.env.example` с `SPRINGRP_POSTER_API_SECRET`.
5. На игровом сервере задать ту же переменную (или `api.secret` в `plugins/SpringPosters/config.yml`).
6. Собрать `spring-poster-client` (`gradlew jar`) и положить JAR в `mods/` внутри `fabric-26.1.2.zip`. Обновить `manifest.json` (`version`, `game.sha256`, `size`).
7. **Полный рестарт** Paper. `/reload` и PlugMan нельзя.
8. После старта: `/ex reload`, если `posters.dsc` обновили на уже работающем сервере.
9. Игроки нажимают Play в лаунчере — скачается новый zip.

Проверка API без секрета должна отвечать `401`. С секретом одинаковый JSON даёт одинаковый `content_hash`.

## Команды

| Команда | Кто | Что делает |
|---|---|---|
| `/poster create` | `marallyzen.poster.create` | Ставит свой `customposter` на вертикальную грань |
| `/poster edit` | владелец / admin | Открывает GUI, глядя на любой вид плаката |
| `/poster clone` | admin | Два раза: источник, затем цель |
| `/poster owner <ник>` | admin | Сменить владельца записи |
| `/poster rerender` | admin | Повторно запечь PNG предпросмотра из БД / кэша |
| `/poster admin unlock` | admin | Снять edit lock |
| `/poster migrate` | admin | Запись БД для уже стоящего плаката |
| `/poster force` | admin | Снять клиентские сессии и вернуть чистые плакаты на стены |
| `/poster remove all` | admin | Сущности + Map bindings + строки БД |

Старые `/poster spawn|give|variant|rebuild` работают как раньше. `/poster spawn` ставит административный плакат.

Shift+ПКМ по плакату открывает тот же редактор. В текстовом диалоге отдельно задаются заголовок, основной текст и подпись автора. Обычный ПКМ — просмотр и анимация.

## Права

Группа «игрок»: `create`, `edit`, `edit.own`, `delete.own`.  
Builder: `poster.admin`, `edit.any`, `delete.any`, `break.admin`.  
`marallyzen.poster.admin` покрывает всё, включая правку административных плакатов.

## Если renderer недоступен

Активная версия не меняется. Черновик в GUI не становится активным. Игрок видит ошибку и может нажать «Сохранить» снова. Сервер не блокируется: HTTP идёт асинхронно. На вылете клиент всё равно рисует текст из сохранённых полей, даже если PNG предпросмотра не обновился.

## Данные

- SQLite: `plugins/SpringPosters/posters.db`
- Кэш PNG/пикселей: `plugins/SpringPosters/cache/<sha256>`
- Audit: `plugins/SpringPosters/audit.log`
- Канал клиента: `springrp:poster` (`session-start` bind / `session-end` unbind)

Не храните PNG в Denizen flags. Секреты, `.env` и `posters.db` в Git не коммитить.
