# Экранные и пространственные эффекты

## Эффекты экрана — SFX

Требуются актуальный клиент SpringRP и право `springrp.sfx` (по умолчанию OP).

```text
/springrp sfx play <игрок> <эффект> <секунды>
/springrp sfx stop <игрок>
```

Длительность — **целое число от 1 до 300 секунд**. Эффект получает указанный игрок; повторный запуск заменяет предыдущий. Завершение времени или `stop` убирает эффект.

Примеры:

```text
/springrp sfx play Akcolotuk vhs 10
/springrp sfx play Akcolotuk rain_glass 20
/springrp sfx stop Akcolotuk
```

Доступные ID:

```text
vhs, shake, chromatic, rain_glass, crt, fog, night_vision, water,
invert, hue_shift, hue_cycle, posterize, sway, shake_heavy, frame_diff,
rgb_split, rain, color_isolate, fog_blue, fog_green, fog_dark, fog_gray,
depth_edges, radar, flashlight, depth_flash, night_vision_pixel,
radial_wave, mirror, vignette, smear, toon_depth, desaturate_distance,
bloom, depth_shift, depth_clip, grayscale_distance, water_caustics,
bright_edges, hue_rotate, channel_mix_rg, channel_mix_bg, glass, blur,
creeper_vision, spider_vision, transparency, entity_outline
```

## Эффекты в мире — VFX

Это отдельная система пространственных эффектов для операторов. Пример:

```text
/springrp vfx list
/springrp vfx play springrp:debug 0.1 10
/springrp vfx stop
```

В `play` после ID идут масштаб **0.01–10** и время **1–60 секунд**. Каталог включает 291 эффект; `list` поддерживает выбор коллекции и страницы. Эффекты видны поддерживаемым клиентам поблизости, в радиусе до 64 блоков.

Основная поддерживаемая конфигурация рендера — Windows x64 и OpenGL. Наличие команды само по себе не гарантирует отображение на любом клиенте и графическом бэкенде.

SFX и VFX можно подключать к [панелям](paneli.md); экранные эффекты также доступны в [событиях кат-сцен](kat-sceny.md).
