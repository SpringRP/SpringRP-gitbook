# Экранные эффекты SpringRP

Команда доступна операторам:

```text
/sfx play <игрок> <эффект> <секунды>
/sfx stop <игрок>
```

Эффекты: `vhs`, `shake`, `chromatic`, `rain_glass`, `crt`, `fog`, `night_vision`, `water`,
`invert`, `hue_shift`, `hue_cycle`, `posterize`, `sway`, `shake_heavy`, `frame_diff`, `rgb_split`,
`rain`, `color_isolate`, `fog_blue`, `fog_green`, `fog_dark`, `fog_gray`, `depth_edges`, `radar`,
`flashlight`, `depth_flash`, `night_vision_pixel`, `radial_wave`, `mirror`, `vignette`, `smear`,
`toon_depth`, `desaturate_distance`, `bloom`, `depth_shift`, `depth_clip`, `grayscale_distance`,
`water_caustics`, `bright_edges`, `hue_rotate`, `channel_mix_rg`, `channel_mix_bg`, `glass`.
Длительность — от 1 до 300 секунд. Эффект отправляется только указанному игроку и автоматически снимается по истечении времени. Повторный запуск заменяет предыдущий эффект; выход с сервера очищает экран вместе с клиентским состоянием.

Для работы нужен обновлённый клиентский SpringPoster и ресурсы `assets/minecraft/post_effect/spring_sfx/`. Клиенты без обновлённого мода не получают визуальный эффект и не должны из-за этого отключаться.

Примеры:

```text
/sfx play neutko vhs 10
/sfx play neutko rain_glass 20
/sfx play neutko night_vision 30
/sfx stop neutko
```
