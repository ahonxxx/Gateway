# Сценарии
Система сценарий позволяет сконфигурировать логику автоматизаций в шлюзе без написания скриптов.

Базовая поддержка появилась в версии **2021.11.21d7**.

## Создание сценариев
На первоначальном этапе создание сценариев возможно ручным созданием файла сценариев.

В дальнейшем будет разработан удобный веб интерфейс где можно легко и быстро создавать сценарии.

## Хранение и загрузка сценариев
Сценарии хранятся в формате json в файле scenes.json

Для загрузки сценариев необзодимо добавить в *init.lua*, добавил *scenes.begin()*. 

Это необходимо на этапе бетатестирования системы, в последующем надобность в этом вызове отпадет и система будет загружаться автоматически.

## Опиcание работы системы
Система сценариев наблюдает за событияеми системы и позволяет реагировать на них действиями.

Условия (триггеры) задаются в блоке *if*.

Действия (экшены) задаются в блоке *then*.

Возможно существование нескольких условия и действий.

Каждое условие и действие имеет свой тип и параметры.

### Типы условий
1. Событие изменение состояния Zigbee устройства

     Параметры: FriendlyName, StateName, Rule, StateValue
    
2. Событие изменение объекта (Пока не реализовано)

     Параметры: ObjName, Rule, ObjValue
     
3. Событие наступления времени (Пока не реализовано)    
4. Событие изменение входа IO (Пока не реализовано)    

### Типы действий
1. Изменение состояния Zigbee устройства

     Параметры: FriendlyName, StateName, Rule, StateValue
    
2. Изменение объекта

     Параметры: ObjName, Rule, ObjValue
     
3. Управление выходами IO (Пока не реализовано)    
4. Отправлка уведомлений в Telegram (Пока не реализовано)  
5. Запуск Lua кода (Пока не реализовано)    
6. Включение/выключение сценария (Пока не реализовано)    
7. Запуск таймера (Пока не реализовано)    

### Правила в параметрах
* Правила для условий: >,<,=,==, ,>=,<=,!=,!,<>

* Правила для действий: =
  
  В дальнейшем можно будет использовать + и - для прибавления и убавления значения

### Логика для условий
В дальнейшем можно будет использовать несколько условий и объединять их логическими операндами.

### Включение и выключение сценария
В дальнейшем можно будет включать и выключать сценарии

## Пример файла сценариев

```
[
  { 
    "desc": "Set 10 brigtness",
    
    "if": [{"t":1, "p": ["room_sw1", "action", "=", "single_left"]}],

    "then": [{"t": 1, "p": ["room_light", "brightness", "=", "10"]}]
  },
  { 
    "desc": "Light off when Obj test_obj is false and Obj test_enable is true",
    
    "if": [
      {"t":2,"p":["test_obj", "=", "false"],"lo":0},
      {"t":2,"p":["test_enable", "=", "true"]}      
    ],

    "then": [{"t": 1, "p": ["room_light", "state", "", "off"]}]
  }
]
```
