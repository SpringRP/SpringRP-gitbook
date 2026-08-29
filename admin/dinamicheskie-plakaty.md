# Динамические плакаты (администратору)

Динамический плакат (`customposter`) — это обычный настенный плакат Marallyzen плюс запечённая карта 128×128. Ресурспак при редактировании не пересобирается.

## Установка

1. Собрать `spring-posters` (`gradlew build`) — JAR попадает в `jar/`.
2. Загрузить `jar/SpringPosters-1.0.0.jar` в `/plugins/` по SFTP.
3. Загрузить `auth-bot/posters-api/` на REG.RU в `www/springrp.ru/posters-api/`.
4. На сайте создать `posters-api/.env` из `.env.example` с `SPRINGRP_POSTER_API_SECRET`.
5. На игровом сервере задать ту же переменную (или `api.secret` в `plugins/SpringPosters/config.yml`).
6. **Полный рестарт** Paper. `/reload` и PlugMan нельзя.
7. После старта: `/ex reload` не обязателен, если `posters.dsc` уже на диске до рестарта.

Проверка API без секрета должна отвечать `401`. С секретом одинаковый JSON даёт одинаковый `content_hash`.

## Команды

| Команда | Кто | Что делает |
|---|---|---|
| `/poster create` | `marallyzen.poster.create` | Ставит `customposter` на вертикальную грань |
| `/poster edit` | владелец / `edit.any` | Открывает GUI, глядя на плакат |
| `/poster clone` | admin | Два раза: источник, затем цель |
| `/poster owner <ник>` | admin | Сменить владельца |
| `/poster rerender` | admin | Повторно запечь из БД / кэша |
| `/poster admin unlock` | admin | Снять edit lock |
| `/poster migrate` | admin | Запись БД для уже стоящего `customposter` |
| `/poster force` | admin | Вернуть анимации на стены, **не** удаляя карту |
| `/poster remove all` | admin | Сущности + Map bindings + строки БД |

Старые `/poster spawn|give|variant|rebuild` работают как раньше.

Shift+ПКМ по плакату открывает редактор. Обычный ПКМ — просмотр и анимация.

## Права

Группа «игрок»: `create`, `edit`, `edit.own`, `delete.own`.  
Builder: `edit.any`, `delete.any`.  
`marallyzen.poster.admin` покрывает всё.

## Если renderer недоступен

Плакат на стене не меняется. Черновик в GUI не становится активным. Игрок видит ошибку и может нажать «Сохранить» снова. Сервер не блокируется: HTTP идёт асинхронно.

## Данные

- SQLite: `plugins/SpringPosters/posters.db`
- Кэш PNG/пикселей: `plugins/SpringPosters/cache/<sha256>`
- Audit: `plugins/SpringPosters/audit.log`
- PDC сущностей: `poster_id`, `poster_role`, `revision`

Не храните PNG в Denizen flags. Секреты, `.env` и `posters.db` в Git не коммитить.
