   Прошивка 7.2 начала принимать входящие команды в формате **JSON** и предусматривает работу модема на скорости **57600**, предыдущая работала на скорости 9600;

Перед обновлением любой версии старше 31.12.2018 до 7.2 необходимо находясь с старой прошивке, в консоли модема отправить команду `AT+IPR=57600;&W`. Если мониторе порта после отправки команды появятся артефакты,что свидетельствует о смене скорости модемом, затем можно обновляться через *.bin файл. Не лишним будет сделать полный сброс настроек `http://192.168.4.1/hardreset`

* [Скачать актуальную версию прошивки 7.2.2 ](https://github.com/martinhol221/SIM800C_ESP8266/tree/master/firmware) управление JSON, обязательно перезалить дашборд
* [FULL - версия дашборд для MQTT Dasch](https://raw.githubusercontent.com/martinhol221/SIM800C_ESP8266/master/daschbord_full.txt) максимум информации

![](https://github.com/martinhol221/SIM800C_ESP8266/blob/master/dashbord/Daschbord.jpg)


## Протокол обмена (MQTT)
Устройство подключается к MQTT броккеру (серверу), и каждую минут публикует в MQTT топик `car/c5/pub/`, на который подписывается смартфон, JSON строку данных `{"pin":[13.73,1,1,0,0,0,1,0],"temp":[14.81,14.50,14.13],"time":["8:54",2day.18:28:24]}`, извлечь необходимый параметр можно с помощью [JSON pach](https://github.com/json-path/JsonPath) доступного в большинстве MQTT приложений

* `$.pin[0]` - вернёт напряжение питания устройства (13.73)
* `$.pin[1]` - вернёт состояние реле `K1` (1) - включено
* `$.pin[2]` - вернёт состояние реле `K2` (1) - включено
* `$.pin[3]` - вернёт состояние реле `K3` (0) - отключено 
* `$.pin[4]` - вернёт состояние реле `K4` (0) - отключено
* `$.pin[5]` - вернёт состояние реле `K5` (0) - отключено 
* `$.pin[6]` - вернёт состояние входа `IN1` (0) - нет напряжения
* `$.pin[7]` - вернёт состояние входа `IN2` (1) - есть напряжение
* `$.temp[0]` - вернёт температуру датчика с индексом `0`  (14.81)
* `$.temp[1]` - вернёт температуру датчика с индексом `1`  (14.50)
* `$.temp[2]` - вернёт температуру датчика с индексом `2`  (14.13)
* `$.time[0]` - вернёт значение таймера обратного отсчета `8:54` минут 
* `$.time[1]` - вернёт время работы устройства `2day.18:28:24`

**Устройство** подписывается на топик `car/c5/sub`, и ожидает команд в JSON формате `{"cmd":[код1,код2,код3,код4]}`, получив команду исполняет последовательность кодов слева направо.

## Коды управления:
#### Коды управления паузами времени
* `1` - Пауза 0.03 сек.
* `2` - Пауза 0.50 сек.
* `3` - Пауза 0.10 сек.
* `4` - Пауза 0.25 сек.
* `5` - Пауза 0.50 сек.
* `6` - Пауза 1.00 сек.
* `7` - Пауза 3.00 сек.
* `8` - Пауза 5.00 сек.
* `9` - Пауза 10.0 сек.

#### Коды управления реле
* `11` - Включить  реле К1 (ПОТРЕБИТЕЛИ, АСС)
* `10` - Отключить реле К1"
* `21` - Включить  реле К2 (ЗАЖИГАНИЕ, ING1)
* `20` - Отключить реле К2
* `31` - Включить  реле К3 (СТАРТЕРА) на время (0.5 - 5сек.)
* `41` - Включить  К4 (другие функции, ACC2, сирена, питание видео-регистратора и др.)
* `40` - Отключить внешнее реле К4
* `51` - Включить  К5 (Обходчик иммобилайзера, ING2)
* `50` - Отключить внешнее реле К5

#### Коды управления расширителем портов на DS2413 по 1-Wire шине 
* `60` - Порт А:0, порт B:0
* `61` - Порт А:1, порт B:0
* `62` - Порт А:0, порт B:1
* `63` - Порт А:1, порт B:1

Общая шина с датчиками температуры DS18B20, временно определение адреса DS2413 не работает

#### Коды установки таймера обратного отсчета

* `70` - Сброс в **00:00**
* `71` - Увеличить значение таймер на **10** секунд 
* `72` - Таймер **+30** секунд 
* `73` - Таймер **+1** минута
* `74` - Таймер **+5** минут
* `75` - Таймер **+10** минут
* `76` - Таймер **+20** минут
* `77` - Таймер **+60** минут
* `78` - Таймер **+6** часов
* `79` - Таймер **+24** часа

Последовательность кодов `...75,73,72...` установит время **00:11:30**, через которое произойдет отключение всех реле K1-K5, если они были включены.

#### Коды контроля
* `81` - Контроль напряжения через 10 секунд, и сравнение с пороговым напряжением
* `82` - Контроль напряжения через 1  мин
* `83` - Контроль напряжения через 5  минут
* `84` - Контроль напряжения через 20 минут
* `90` - Снятие с охраны (игнорирование импульсов с входа IN2)
* `91` - Постановка на охрану (контроль входа IN2)
* `92` - Звонок хозяину на номер заданный через Web-интерфейс
* `93` - Завершение вызова
* `94` - SMS уведомление через 10 секунд.          
* `95` - SMS уведомление через 5 минут.
* `99` - Активировать Wi-Fi точку доступа на устройстве на 20 минут

#### Коды отправки данных в MQTT
* `100` - Запросить текущие параметры по MQTT в приложение (обновить)
* `101` - Запросить уровень GSM сигнала, `car/c5/pub/debug` `{"rssi":[XX,0,0,0]}`         
* `102` - Запрос локации, возвращает в топик `car/c5/pub/gps` `52.834681,21.694131`.
* `103` - USSD запрос #100#
* `104` - USSD запрос *100# вернется в топик `car/c5/pub/ussd` в DPU или GSM кодировке
* `105` - USSD запрос *102#
* `106` - USSD запрос *105#   

## Контроль состояния входов IN1 и IN2

Как првило вход `IN1` или `IN2` подключается на выход концевика педали тормоза (для АКПП), или на датчик включенной передачи (МКПП), и может разрешать включение реле при дополнительных кодах управления реле:

* `X2` Включить реле **X** в случае если на IN1 напряжение равно 0в.
* `X3` Включить реле **X** в случае если на IN1 напряжение больше 5в.
* `X4` Включить реле **X** в случае если на IN2 напряжение равно 0в.
* `X5` Включить реле **X** в случае если на IN2 напряжение больше 5в.
* `X6` Включить реле **X** в случае если напряжение АКБ больше порогового.
* `37` Включить  реле К3 (СТАРТЕРА) на время заданное в настройках WEB-интерфейса устройства, при условии, если на входе IN1 нет напряжения, а напряжение АКБ меньше заданного через Веб-интерфейс, напряжения зарядки.

где `X` - номер реле от 1 до 5 (K1...K5)

## Контроль по аппаратному прерыванию входов IN1 и IN2

* При появлении импульса выше + 5в. на IN1 (концевик педали тормоза)  происходит отключение всех реле.
* При смене логического состояния на IN2 (датчик удара, датчик аварийного давления масла) осуществляется исполнение сценария ТРЕВОГА

**********
## Контроль температуры

Запрос `{"temp_min":4.25,"temp_max":18.50,"termostat":X}` устанавливает значение минимальной температуры для включения реле `4.25` град, максимальной температуры для отключения реле `18.50`, где  `X` - режим работы термостата

* `0` - контроль температуры отключен
* `1` - включено управление на реле K1
* `4` - включено управление на реле K4
**********

## Контроль местоположения

Модуль не имеет собственного GPS приемника, координаты рассчитываются триангуляцией по базовым станциям, подобно локации в смартфонах, и имеют точность 50 - 800 м. в условиях города. В целях защиты от несанкционированного использования функция не доступна в готовых устройствах, и становится доступной только после обновления прошивки с Github, при условии отсутствия злого умысла в ее применении, и недопустимости слежения за кем-то.

## Активация WI-FI точки доступа

Устройство генерирует точку доступа SSID `Webasto_123456` на 600 секунд с паролем `martinhol221` , к которой можно подключиться с телефона для обновления прошивки или внесения изменений в настройки, страница настроек доступна по адресу `http://192.168.4.1`.
Активация WIFI возможна несколькими способами

* Запрос из приложения `{"cmd":[99]}`
* Удержание кнопки подключенной между `SCL` и `GND` более 3 сек.
* Снятие и подача питания на устройство

![](https://github.com/martinhol221/SIM800C_ESP8266/blob/master/dashbord/web.jpg)

По прошествии 10 минут WiFi отключается и устройство уходит в энергосберегающий режим потребляя ток 17-22 мА.

## Управление по звонку с вводом DTMF команд

При звонке на устройство модем снимет вызов только с телефона хозяина, после "снятия трубки" будет ожидать ввода команд

* `1*1234` - Исполнит сценарий СТАРТ заданный через страницу **Алогоритмы** ВЕБ-интерфейса
* `0`      - Исполнит сценарий СТОП 
* `555`    - Отправит SMS через 10 сек. 
* `777`    - Перезагрузит модем. 
* `#`      - удалит введенные символы

где `1234`     - пинкод для управления   

Сценарии **СТАРТ**, **СТОП** и **ТРЕВОГА** задаются через WEB-интерфейсе, и сохраняются в памяти устройства

![](https://github.com/martinhol221/SIM800C_ESP8266/blob/master/dashbord/scenariy2.jpg)

****

## Выбор MQTT приложения в телефон для управления по GPRS

Для Android

* [MQTT Dash Google Play](https://play.google.com/store/apps/details?id=net.routix.mqttdash&hl=ru) рекомендую
* [IoT MQTT Panel](https://play.google.com/store/apps/details?id=snr.lab.iotmqttpanel.prod)
* [IoT MQTT Dashboard](https://play.google.com/store/apps/details?id=com.thn.iotmqttdashboard)
* [Linear MQTT Dashboard](https://play.google.com/store/apps/details?id=com.ravendmaster.linearmqttdashboard)

Для IOS: 
[IoT OnOff](https://itunes.apple.com/be/app/iot-onoff/id1267226555?mt=8) не обробовано

## Выбор MQTT брокера (далее сервера)

***********************************

MQTT broker

https://www.cloudmqtt.com

http://flyhub.org

**********************************

**Главное окно программы MQTT Dash** 
![](https://github.com/martinhol221/SIM800L_MQTT/raw/master/other/17.jpg)


### Регистрация на https://www.cloudmqtt.com/

* зарегистрируйтесь и залогинтесь

* нажмите  **+Create New Instanct**

* Name **любое значение**

* Plan  **Cute Cat (Free)**

* Data center **EU-West-1 (Ireland)**

* Tags **любое значение**

* Нажать по **Create New Instance**

* Перейти в созданный профиль

Server

User

Password

Port

перенести значение этих параметров в  MQTT Dash, [Скачать](https://play.google.com/store/apps/details?id=net.routix.mqttdash&hl=ru), можно использовать любой другой MQTT клиент

**Настройка приложения к серверу**

![](https://github.com/martinhol221/SIM800L_MQTT/raw/master/other/mqtt-9.jpg)

**Загрузка настроек интерфейса**

Открыть в приложении ***Импорт/экспорт*** (1)

Нажать ***Подписаться и ждать метрики*** (2) 

![](https://github.com/martinhol221/SIM800L_MQTT/blob/master/other/setupMqtt.JPG)

В вебсокете https://api.cloudmqtt.com/console/________/websocket  заполнить  и отправить настройки в телефон

![](https://github.com/martinhol221/SIM800L_MQTT/blob/master/other/setupMqtt2.JPG)

***Topic***  metrics/exchange

Сcылки на конфиг файлы выше, если не понятно смотрим нудное видео без звука 

[Видео по регистрации на youtube.com](https://www.youtube.com/watch?v=xgZZ417HFFQ)

************************************

## Готовая приборная панель программы MQTT Dasch (далее daschbord)

![](https://github.com/martinhol221/SIM800C_ESP8266/blob/master/dashbord/daschbord1.jpg)


**********
## Настройки плиток в MQTT Dash в ручном режиме

* Тип `Текст`
* Имя `Температура`
* Топик (SUB) `car/c5/pub` - желательно не изменять
* Извлечь, используя JSON path.... `$.temp[0]` - для первого термодатчика
* Другие настройки `QoS(0)`, цвет и размер плитки по вкусу.

***

* Тип `Диапазон/прогресс`
* Имя `Напряжение`
* Топик (SUB) `car/c5/pub` - желательно не изменять
* Извлечь, используя JSON path.... `$.pin[0]` - для чтения напряжения
* Минимум `0`, Максимум `15`, Постфикс `V`, Точность `2`
* Плитка мигает для привлечения внимания... `val > 13.2`
* Другие настройки `QoS(0)`, цвет и размер плитки по вкусу.

**********

* Тип `Переключатель/кнопка`
* Имя `СТАРТ`
* Топик (SUB) `car/c5/pub` - желательно не изменять
* Извлечь, используя JSON path.... `$.pin[2]` - контроль по состоянию реле зажигания
* Вкл: `1`, Выкл: `0`, другие настройки `QoS(0)`
* Во вкладку `ON TAP` должен быть помещен этот код , который исполняется при нажатии

> app.publish('car/c5/sub','{"cmd":[51,5,22,5,12,7,36,7,50,5,10,5,100,82]}', false, 0);

**********


* Тип `Переключатель/кнопка`
* Имя `СТОП`
* Топик (SUB) `car/c5/pub` - желательно не изменять
* Извлечь, используя JSON path.... `$.pin[2]` - контроль по состоянию реле зажигания
* Вкл: `1`, Выкл: `0`, другие настройки `QoS(0)`
* Во вкладку `ON TAP` должен быть помещен этот код , который исполняется при нажатии

> app.publish('car/c5/sub','{"cmd":[50,4,20,4,10,70,100]}', false, 0);



### Потребление трафика:

Один пакет GPRS данных состоит в среднем из 100 байт, отправляется устройством каждую минуту, 100 байт х 60 минут х 24 часа х 31 день = около 4.5 MB/мес, однако на практике оператор трафик считают больший, у меня набегает 9.5 Мб в месяц.
Стоит отметить что сотовые операторы по разному округляют трафик в конце сессии, в результате чего, списывать с баланса могут в разы больше фактически затраченного.

![](https://github.com/martinhol221/SIM800L_MQTT/blob/master/other/trafic.JPG)
