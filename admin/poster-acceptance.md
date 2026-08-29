# Приёмочные тесты динамических плакатов

Дата: 2026-08-29. Автоматические тесты: `spring-posters` JUnit (HMAC, wrap, hash, PNG, map colors) и `auth-bot/posters-api/tests/run.php`.

Ручной прогон на live (`5.129.240.121:25567`) после рестарта:

| # | Сценарий | Ожидание | Статус |
|---|---|---|---|
| 1 | Игрок `/poster create`, текст в книге, Сохранить | Плакат на стене с текстом | pending live |
| 2 | Два других игрока смотрят без релога и без нового ресурспака | Тот же текст | pending live |
| 3 | Рестарт сервера | Изображение на месте | pending live |
| 4 | Unload/load чанка | Карта восстанавливается | pending live |
| 5 | Три разных головы | Все три на рендере | pending live |
| 6 | Несуществующий ник | Ошибка в GUI, редактор жив | pending live |
| 7 | Длинный текст | Перенос, не вылезает за 85px | covered by wrap tests |
| 8 | Два редактора сразу | Второй видит «редактирует &lt;ник&gt;»; revision check | pending live |
| 9 | Чужой плакат обычным игроком | Отказ | pending live |
| 10 | Admin `edit.any` | Открывает любой | pending live |
| 11 | Renderer выключен | Старая картинка, ошибка, повтор | pending live |
| 12 | `/poster force` | Карта на месте, без дублей | pending live |
| 13 | `/poster remove all` | Нет сущностей, нет строк БД | pending live |
| 14 | Повторное сохранение | Тот же `map_id`, revision++ | unit: hash reuse |
| 15 | Предметы GUI | Курсор не теряет стек, книга не дюпается | pending live |
| 16 | Static posters | Без визуальных изменений | pending live |
| 17 | Ванильный клиент без Fabric | Карта видна | pending live |
| 18 | Git | Нет паролей, `.env`, db, generated PNG | checked before push |

Секреты в diff проверяются поиском паролей, `.env`, API secret values и служебных URI — без коммита `.cursor/rules`, `.env` и `posters.db`.
