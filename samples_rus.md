# Примеры типовых сценариев

## Включение освещения по датчику движения и выключение через определенное время
Такой сценарий легко решается без использования lua скриптов. Разберем пример из датчка движения Aqara lumi.sensor_motion и исполнительного реле на 8 каналов с [modkam](https://modkam.ru/?p=1638)


### Вариант 1 с использованием команд [SimpleBind](/simplebind_rus.md)

На вкладке датчика движения настраиваем при включении occupation=true будет включать освещение, при occupation=false соответственно выключать.

Пример команды SimpleBind для включения реле по датчику движения:
```
true,0x00124B001F7CA144,state_l1,ON;false,0x00124B001F7CA144,state_l1,OFF;
```
![](/img/ocup_sb.png)

С помощью occupancy_timeout можно задать интервал, в течении которого будет сбрасываться значение датчика. 


### Вариант 2 с использованием значений датчика освещения 
В данном примере мы будем использовать датчик движения Aqara 	lumi.sensor_motion.aq2 со встроенным датчиком освещенности  и исполнительное реле на 8 каналов с [modkam](https://modkam.ru/?p=1638). 

Создаем сценарий occupation.lua

```
local state =  zigbee.value(tostring(Event.ieeeAddr), "occupancy")
local lightlevel =  zigbee.value(tostring(Event.ieeeAddr), "illuminance")
local minlightlevel=300 --зададим минимальный уровень освещенности, когда необходимо включать освещение

if (state) then telegram.send("Датчик движения ".. Event.FriendlyName  .." обнаружил активность") 
  
if (lightlevel)<minlightlevel then 
  zigbee.set("0x00124B001F7CA144", "state_l2", "ON")
  telegram.send("Свет в комнате ".. Event.FriendlyName  .." включили") 
end    
    
    
 else
  telegram.send("Значение датчика движения "..  Event.FriendlyName .."  нормализовалось") 
  
  if (lightlevel)<minlightlevel then 
  zigbee.set("0x00124B001F7CA144", "state_l2", "OFF")
  telegram.send("Свет в комнате ".. Event.FriendlyName  .." выключили") 
end    
end 
```
На вкладке датчика движения привязываем сценарий 

![](/img/ocup_lua2.png)



### Вариант 3 с использованием астротаймера

В данном примере мы будем использовать любой датчик движения, а в качестве определения необходимости включения освещения будем использовать астро-таймер, встроенный в SLS шлюз. Управлять будет также  реле на 8 каналов с [modkam](https://modkam.ru/?p=1638).  Перед настройкой астротаймера, не забудьте в настройках системы указать ваши коодинаты:

![](/img/astrosettings.png)

Создаем сценарий occupation_astro.lua

```
local state =  zigbee.value(tostring(Event.ieeeAddr), "occupancy")
local lightlevel =  zigbee.value(tostring(Event.ieeeAddr), "illuminance")
local minlightlevel=300 --зададим минимальный уровень освещенности, когда необходимо включать освещение

if (state) then telegram.send("Датчик движения ".. Event.FriendlyName  .." обнаружил активность") 
  
if (lightlevel)<minlightlevel then 
  zigbee.set("0x00124B001F7CA144", "state_l2", "ON")
  telegram.send("Свет в комнате ".. Event.FriendlyName  .." включили") 
end    
    
    
 else
  telegram.send("Значение датчика движения "..  Event.FriendlyName .."  нормализовалось") 
  
  if (lightlevel)<minlightlevel then 
  zigbee.set("0x00124B001F7CA144", "state_l2", "OFF")
  telegram.send("Свет в комнате ".. Event.FriendlyName  .." выключили") 
end    
end 
```