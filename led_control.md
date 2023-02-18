# LED

SLS шлюз может управлять различными  светодиодами. Это может быть подсветка корпуса Xiaomi, круглые платы с modkam.ru, либо индикационный многофункциональный светодиод шлюза SLS DIN MINI. В дополнение можно подключить обычную или адресную светодиодную ленту.

Управлять LED можно скриптами [LUA](/lua_rus.md), [HTTP API](/http_api_rus.md) и MQTT. Можно задавать яркость и цвет свечения, а также включать и выключать различные [эффекты](/led_control.md#таблица-эффектов)

[Примеры для различных устройств](/samples_rus.md#Управление-адресными-светодиодами)

## Режимы

- ON - включен
- OFF - выключен
- AUTO - оповещение шлюза о своем состоянии по следующему алгоритму:
  - зеленый горит - Join
  - синий переливается - режим AP
  - синий мигает - идет подключение к Wi-Fi
  - красный - ошибка подключения к Wi-Fi
  - белый горит - удержание сервисной кнопки в момент старта шлюза

## Управление

### LUA

#### os.led()

```lua
os.led(mode, brightness, r, g, b[, effect])
-- mode - STR, режим. OFF - выключено, ON - включено, AUTO - индикация режимов/состояний шлюза (см. описаниее далее) 
-- brightness - INT, яркость (целое, от 0 до 255)
-- r, g, b - INT, цвет (целое, от 0 до 255 или -1, если цвет менять не требуется)
-- effect - INT, включает эффекты в соответствии с таблицей ниже
```

### HTTP API

#### Получить текущий статус

```http
GET /api/led
```

```json
{
    "state":"OFF",
    "brightness":255,
    "color":{"r":0,"g":0,"b":0},
    "color_mode":"rgb",
    "mode":"auto"
}
```

- `state` - STR, статус светодиода - `ON`/`OFF`
- `brightness` - INT, яркость
- `color` - объект со значениями цвета `RGB`
- `color_mode` - цветовая модель (для совместимости с HA)
- `mode` - режим (`AUTO`, `ON`, `OFF`)

#### Управление LED

Управление LED осуществляется передачей JSON объекта:

```http
GET /api/led?set=JSONObject
```

JSON объект:

```json
{
    "mode": "on",
    "brightness": 100,
    "color":{
        "r":255,
        "g":150,
        "b":200
    },
    "effect":10
}
```

Любой из параметров `mode`, `brightness`, `color`, `effect` может отправляться как по отдельности, так и в любой подходящей под задачу комбинации в одном запросе.

В ответ возвращается [текущий статус](/http_api_rus.md#получить-текущий-статус):

## Примеры

### HTTP API. Включить LED с красным цветом и яркостью 50%

```http
GET /api/led?set={"brightness":128,"color":{"r":255,"g":150,"b":200}}
```

### HTTP API. Включить LED с эффектом 10 и яркостью 50%

```http
GET /api/led?set={"brightness":128,"effect":10}
```

### LUA. Включить LED с красным цветом и яркостью 100%

Вариант 1

```lua
os.led("ON", 255, 255, 0, 0)
```

Вариант 2

```lua
local led = {mode = "ON", brightness = 255 , r = 255, g = 0, b = 0}
os.led(led.mode, led.brightness, led.r, led.g, led.b)
```

### LUA. Вернуть в режим AUTO

```lua
os.led("AUTO")
```

## Таблица эффектов

| **Номер** | **Краткое описание** | **Цвет** | **Действие** | **Eng** |
|-|-|-|-|-|
| 0 | Постоянное свечение (эффекты отключены)| моно |Статика |Static|
|1 |Частые вспышки (период свечения больше чем период затемнения)| моно |Вспышка| Blink                                                        |
|2 |Смена яркости от минимума до максимума и обратно (20сек) |моно |Дыхание |Breath                                                            |
|3 |Зажигает последовательно круг и гасит по часовой стрелке |моно| Стирание цвета| Color Wipe                                                 |
|4 |Гасит круг по часовой и зажигает по часовой стрелке |моно| Инвертированное стирание цвета |Color Wipe Inverse                              |
|5 |Зажигает круг по часовой и гасит против часовой |моно |Реверсивное стирание цвета |Color Wipe Reverse                                      |
|6 |Гасит круг по часовой и зажигает против часовой| моно| Реверсивное инвертированное стирание цвета| Color Wipe Inverse Reverse              |
|7 |Зажигает по часовой перебирая цвета| RGB| Стирание случайного цвета| Color Wipe Random                                                     |
|8 |Случайная смена цвета через 0,5сек. |RGB |Случайный цвет |Random Color                                                                     |
|9 |Рандомно меняется цвет одной точки с повышенной яркостью через 0.5 сек|RGB |Одиночный динамический| Single Dynamic                        |
|10| Рандомно меняется цвет половины точек через 0.5 сек| RGB |Мульти динамический|Multi Dynamic                                              |
|11| Заполняет круг одним цветов, который плавно изменяется по радужной палитре| RGB| Радуга |Rainbow                                          |
|12| Радуга, меняющаюся от начала к концу |RGB| Циклическая радуга| Rainbow Cycle                                                              |
|13| Бегающая точка по сектору 360гр.|моно |Сканер |Scan                                                                                      |
|14| Две бегающие точки по секторам 180гр.| моно| Двойной скане|р Dual Scan                                                                    |
|15| См. 2 только 10 сек. |моно |Затухание |Fade                                                                                               |
|16| 1 святящаяся и 2 черных точки, вращаются по часовой стрелке| моно |Театральное преследование |Theater Chase                               |
|17| См. 16 со сменой цвета из политры радуги |RGB |Радужная театральное преследование |Theater Chase Rainbow                                  |
|18| Бегущая градиентная тень из ~10% точек |моно |Бегущие огни |Running Lights                                                                |
|19| Произвольные вспышки точек (1сек) |моно |Мигание |Twinkle                                                                                 |
|20| Произвольные вспышки цветных точек (1сек) |RGB| Цветное мигание |Twinkle Random                                                           |
|21| Произвольные вспышки точек с плавным затуханием яркости (0,5сек)| моно| Мигание с затуханием| Twinkle Fade                                |
|22| Произвольные вспышки цветных точек с плавным затуханием яркости (0,5сек)| RGB| Цветное мигание с затуханием |Twinkle Fade Random          |
|23| Произвольные вспышки точек (0,2сек)| моно |Искра |Sparkle                                                                                 |
|24| Произвольная белыя вспышка (0,2сек) на выбранном фоне |RGB |Вспышка искры| Flash Sparkle                                                  |
|25| Произвольные множественные белые вспышки на выбранном фоне |RGB| Гипер вспышки| Hyper Sparkle                                             |
|26| Частые вспышки (период свечения меньше чем период затемнения) |моно |Стробоскоп |Strobe                                                   |
|27| Частые вспышки со сменой цвета из политры радуги (период свечения меньше чем период затемнения) |RGB| Радужный стробоскоп |Strobe Rainbow |
|28| Несколько последовательных вспышек с паузой 2 секунды |моно| Мульти стробоскоп| Multi Strobe                                              |
|29| Частые вспышки со сменой цвета из политры радуги (период свечения больше чем период затемнения) |RGB |Радужная вспышка| Blink Rainbow     |
|30| Заполняет круг белым и двигает 2 точки выбранного цвета по часовой |RGB| Преследование на белом| Chase White                              |
|31| Заполняет круг выбранным цветом и двигает 2 белые точки по часовой |RGB |Преследование на цветном| Chase Color                            |
|32| Заполняет круг разными цветами по очереди и двигает 2 белые точки по часовой |RGB| Преследование на случайном цвете| Chase Random         |
|33| Зажигает радугу по кругу и двигает 2 белые точки по часовой| RGB| Преследование на радуге |Chase Rainbow                                  |
|34| Заполняет круг выбранным цветом и двигает 2 белые точки по часовой вспышками |RGB |Преследование со вспышкой |Chase Flash                 |
|35| Двигает 2 белые точки по часовой вспышками заполняя круг всеми цветами| RGB |Цветное преследование со вспышкой| Chase Flash Random        |
|36| Зажигает круг белым цветом и двигает 2 радужные точки по часой| RGB| Радужное преследование со вспышкой |Chase Rainbow White              |
|37| Зажигает круг выбранным цветом и двигает тень в 2 точки по часовой| моно| Преследование затемнения| Chase Blackout                        |
|38| Двигает тень в 2 точки по часовой по радужному кругу| RGB |Радужное преследование затемнения | Chase Blackout Rainbow                      |
|39| Зажигает круг по часовой одним цветом и меняет его при движении в обратную сторону |RGB |Толкание цвета| Color Sweep Random               |
|40| Бегущая точка выбранного цвета с хвостом из точек |моно| Бегущий цвет | Running Color                                                      |
|41| Быстро бегущие красные и синии точки |RGB |Бегущий красно синий| Running Red Blue Хорошо заметно если предыдущий эффект был со сплошным фоном|
|42| Быстро бегущие точки произвольного цвета |RGB |Бегущий случайный цвет |Running Random См. 41                                              |
|43| Бегущая туда-обратно точка выбранного цвета с хвостом (6 точек с затуханием яркости)| моно |Сканер Ларсона| Larson Scanner                |
|44| Бегущая в одну сторону точка выбранного цвета с хвостом (6 точек с затуханием яркости)| моно| Комета| Comet                               |
|45| Возникающая в случайном месте вспышка выбранного цвета с обрамлением из 5-6 точек с затухающей яркростью| моно| Фейерверк| Fireworks      |
|46| См. 45, но цвет случайный |RGB |Случайный фейерверк |Fireworks Random                                                                     |
|47| См. 41, но красно - зеленая гамма |RGB |С Рождеством!| Merry Christmas                                                                    |
|48| Зажигает круг выбранным цветом в половину ярости и зажигает в случайном порядке точки с яркостью от 50 до 100% |моно| Мерцание огня |Fire Flicker |
|49| См. 48, но менее контрастный (случаные точки от 50 до 70% яркости)| моно| Мерцание огня (мягкое)| Fire Flicker (soft)                     |
|50| См. 48, но более контрастный (начальная яркость 30%) |моно| Мерцание огня (интенсивное) |Fire Flicker (intense)                           |
|51| Заполняет круг парами красных и белых точек, которые меняются местами| RGB |Цирк сгорел |Circus Combustus                                 |
|52| См. 47, но оранжево-синяя гамма |RGB |Хэллоуин| Halloween                                                                                 |
|53| Бегущие 2 черные точки запонлняют круг выбранным цветом |моно |Двуцветное преследование| Bicolor Chase                                    |
|54| Заполняет круг шаблном 2 черные точки и точка выбранного цвета, и двигает по кругу|моно |Трехцветное преследование |Tricolor Chase       |
|55| Заполняет круг выбранным цветом и в произвольном порядке за 2 секунды меняет яркость у половины точек| моно| Мигание| FOX TwinkleFOX      |
|56| Вспышка с затуханием выбранным цветом. Сначала все точки, а через 10 секунд половина точек| моно |Дождь| Rain                             |
|57| Сердечный ритм, цвет меняется из радужной палитры| RGB| Биение сердца| Bpm                                                                |
|58| Градиет от начала до конца с плавными переходами между цветами |RGB |Радуга 2| Rainbow 2                                                  |
|59| На темном фоне зажигает 30% разноцветных точек с плавным угасанием |RGB| Конфетти| Сonfetti                                               |
|60| См. 58, но добавляется случайная белая вспышка с разным интервалом |RGB |Радуга с блесткой |Rainbow with glitter                          |
|61| Реверсивный бегущий огонь с радужной палитрой на всю длину |RGB||Sinelon Sinelon                                                         |
|62| 6 бегущих точек с переливом цветов с плавным затуханием |RGB| Жонглирование |Juggle                                                       |