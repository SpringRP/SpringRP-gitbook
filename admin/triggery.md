# Области-триггеры

SpringTriggers запускает команды при входе игрока в прямоугольную область и выходе из неё. Области сохраняются после перезапуска. Право настройки — `springtrigger.admin`.

## Создание

```text
/springtrigger wand
```

Жезлом нажми ЛКМ по одному углу, ПКМ — по противоположному. Выделяется весь объём между блоками, включая границы. Максимальный размер по умолчанию — **20 × 20 × 20 блоков**.

```text
/springtrigger create lobby
/springtrigger addcommand lobby enter springrp platforms open lift 3s
/springtrigger addcommand lobby exit springrp platforms close lift 3s
```

В примере должна заранее существовать платформа `lift`. Внутренняя команда вводится **без начального `/`** и выполняется от консоли.

## Подстановки

| Подстановка | Значение |
| --- | --- |
| `{player}` | Ник вошедшего или вышедшего игрока |
| `{uuid}` | UUID игрока |
| `{world}` | Мир |
| `{x}`, `{y}`, `{z}` | Координаты |

Действие срабатывает один раз при пересечении границы, а не каждый тик внутри области. Если вернуться, вход сработает снова. Персональный КД и одноразовый просмотр кат-сцен настраиваются в [редакторе сцен](kat-sceny.md).

## Управление

```text
/springtrigger list
/springtrigger info lobby
/springtrigger removecommand lobby enter 1
/springtrigger clearcommands lobby exit
/springtrigger delete lobby
/springtrigger reload
```

Номер команды берётся из `info`, отсчёт начинается с **1**. `clearcommands` принимает `enter`, `exit` или `both`. `delete` удаляет область и её действия.

Для кат-сцен удобнее привязывать область через меню сцены: оно сохраняет остальные команды и позволяет снять только свою привязку.
