# Предметы, виноград и диктофоны

Команды ниже предназначены для администрации. Они помогают проверить механику без изменения клиентской сборки.

## Каталог предметов

Право: `springrp.customitems.admin`.

```text
/springrp customitems springrp keyring
/springrp customitems springrp key
/springrp customitems springrp lock
/springrp customitems vinery grapes 8
```

Tab показывает категории и предметы. Наличие декоративного предмета в каталоге не гарантирует отдельную механику или рецепт.

## Виноград

Право: `springrp.grapes.admin`. Смотри на подходящую листву в пределах шести блоков:

```text
/grapes status
/grapes grow
/grapes remove
```

`grow` создаёт гроздь для проверки, `remove` убирает целевую гроздь. После сбора проверь исчезновение модели и количество предметов, включая подбор при уже полном стаке.

## Фантомные предметы

Право: `springrp.phantomitems.admin`.

```text
/springrp pitems list
/springrp pitems spawn <id,id,...> [количество|each] [радиус]
/springrp pitems remove [радиус]
```

Используй ID материалов, например `/springrp pitems spawn diamond,emerald each 1`. `list` показывает уже созданные кучи. Предметы — декорации, игроки не могут их подобрать. Удаление выполняется вокруг администратора; для одной кучи выбирай небольшой радиус. `purge` является массовой очисткой.

## Диктофон с тестовым звуком

Нужны права `springrp.dictaphone.use` и `springrp.dictaphone.admin`. Посмотри на верх твёрдого блока в пределах шести блоков:

```text
/springrp dictaphone spawn test_voice
/springrp dictaphone info
/springrp dictaphone remove
```

Первые две команды создают и проверяют тестовый диктофон, последняя удаляет тот, на который направлен взгляд. `list` показывает установленные устройства. Массовые `remove all` и удаление по радиусу затрагивают несколько устройств.

Звук хранится **на сервере** в `plugins/SpringDictophone/recordings/`. Для новой записи положи пару файлов с одинаковой основой имени:

```text
interview.ogg
interview.yml
```

OGG должен быть **Vorbis**. Пример YAML:

```yaml
lines:
  - speaker: Спикер
    audio: interview
    text: "Текст реплики."
```

Значение `audio` должно совпадать с именем файла без расширения. Длительность берётся из самого аудио. Затем создай устройство через `/springrp dictaphone spawn interview`. Пересобирать ресурспак для новой записи не нужно.

Диктофон передаёт запись открывшему его игроку через Plasmo Voice. Если звука нет, проверь мод, громкость источника, пару файлов и кодек.

## Усиление броска

`/throwables strength` показывает текущий максимум. `/throwables strength 2.0` меняет его; нужны права `springthrowables.admin`. Допустимый диапазон — **1–6**, значение по умолчанию — **2**.
