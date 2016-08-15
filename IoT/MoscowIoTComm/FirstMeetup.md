Первая встреча Moscow IoT Community: как мы нашли баг в Visual Studio


![enter image description here](https://habrastorage.org/files/8e9/f18/e18/8e9f18e188524602a5cb6488d4ce7b0b.jpg)

Недавно прошла первая встреча [Московского сообщества Интернета Вещей](http://www.meetup.com/Moscow-Internet-of-Things-Community/) целью которой было пройти путь от создания простого устройства на Raspberry Pi до визуализации данных полученных в облаке от этого устройства. Было интересно и весело, а еще мы нашли баг в Visual Studio. Подробности под катом (много фото).

----------

Сообщество у нас молодое, но активное. [Первая встреча](http://www.meetup.com/Moscow-Internet-of-Things-Community/events/232800557/) получилась масштабной - полноценная однодневная лабораторная работа. Партнером встречи выступила компания [Microsoft](https://habrahabr.ru/company/microsoft/) предоставив нам помещение и пропитание. Ну а "хедлайнером" встречи выступил [Каталин Георгиу](https://mvp.microsoft.com/ru-ru/spotlight/user/4030788) - опытнейший разработчик из Румынии, 6 раз подряд получивший награду Microsoft MVP.  Он организует и проводит подобные лабораторные работы на территории Восточной Европы. Ну а на этот раз он стал нашим гостем:

![enter image description here](http://photos1.meetupstatic.com/photos/event/8/6/9/7/600_453034455.jpeg)

Итак, у участников были [наборы](https://developer.microsoft.com/en-us/windows/iot/Docs/AdafruitMakerKit.htm) из Raspberry PI 2, проводов кнопок светодиодов и всего остального, необходимого для работы.

Целью работы было:

 - Установить на Rasbpberry Pi2 Windows 10 IoT Core 
 - Подключить светодиод и помигать им 
 - Добавить кнопку и сделать так, чтобы светодиод включался по нажатию кнопки 
 - Замерять время 
 - Отправить измеренное время в Azure через Azure IoT Hub
 - Визуализировать данные с помощью Power BI

Для того, кому трудно сделать все самостоятельно, уже существует множество готовых [примеров](https://developer.microsoft.com/ru-ru/windows/iot/samples).

Работа проходила в очень дружеской и творческой атмосфере:

![enter image description here](http://photos3.meetupstatic.com/photos/event/8/6/5/d/600_453034397.jpeg)

Желание научиться новому объединило совершенно разных людей

![enter image description here](http://photos1.meetupstatic.com/photos/event/8/6/5/a/600_453034394.jpeg)

Ну а "творческий беспорядок" добавлял процессу работы необходимый антураж :)

![enter image description here](http://photos1.meetupstatic.com/photos/event/8/6/5/c/600_453034396.jpeg)

Кроме занятия лабораторными работами, на встрече можно было послушать доклады про [.Net Micro Framework](https://geektimes.ru/search/?q=%5B.net%20micro%20framework%5D&target_type=posts) и Azure IoT Hub ну и, конечно, пообщаться с докладчиками и экспертами.

![enter image description here](http://photos3.meetupstatic.com/photos/event/8/6/9/4/600_453034452.jpeg) 

Стоит сказать что в процессе работы многие столкнулись с проблемами при подключении Visual Studio к Rasbpbery Pi2 с Windows 10 IoT Core.
В анонсе встречи была просьба установить все [необходимые программы и компоненты](https://developer.microsoft.com/en-us/windows/iot/docs/kitsetuppcrpi) однако не у всех все прошло гладко.

Как оказалось, проблема была в русском языке Visual Studio. Русская версия искала библиотеки, необходимые для работы с Windows 10 IoT Core, в папке для локализованных библиотек. Тогда как все необходимое было только в "англоязычной" папке. Немного помучившись, было найдено решение. Нужно было просто установить английский языковой пакет. Однако сам процесс установки пакета оказался долгим и из-за этого не все пришедшие успели выполнить всю лабораторную работу.

Но с другой стороны, кое кому удалось даже перевыполнить план и попробовать все "железки" входящие в комплект:

![enter image description here](https://habrastorage.org/files/3fd/f58/7ff/3fdf587ff8374384bf3b379d4a5ece3e.jpg)

Ну а следующая встреча [Moscow IoT Community](http://www.meetup.com/Moscow-Internet-of-Things-Community/) уже не за горами. Присоединяйтесь и ждите анонсов! Будет интересно!
