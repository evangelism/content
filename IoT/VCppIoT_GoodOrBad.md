

Visual C++ for IoT Development: Прорыв или разочарование?
=========================================================

![enter image description here](https://habrastorage.org/files/a26/acc/d5b/a26accd5b8fe481387bf227d97854eaa.png)

Как известно мир не стоит на месте и активно развивается. Особенно в сфере IT и конкретно разработки ПО. Самое трудное в этом деле это уследить за всеми новинками и выхватить наиболее ценные и полезные из них. Вот и я только недавно наткнулся на дополнение к Visual Studio под названием Visual C++ for IoT Development, которое существует уже почти год. Потенциально - очень полезная штука, но пока это больше яркое название, чем настоящее Visual C++ для IoT. Давайте разберемся почему.


----------

Идея использовать Visual Studio при работе с мироконтроллерами посещает меня давно и регулярно. А в последнее время эта идея превратилась в практику. Все мои проекты давно плотно интегрированы с Visual Studio. Первое время я использовал ее просто как средство взаимодействия с системами контроля версий. Потом начал частично писать в ней код, а теперь еще и использую встроенную в нее систему Unit тестов. Но компиляция и отладка всегда оставались где-то за бортом.

Увидев заголовок Visual C++ for IoT Development, я подумал, что век мучений закончился! Хватит писать код в одном месте, а компилироать и отлаживать в другом постоянно переключаясь то туда, то сюда. Появилась надежда все делать в Visual Studio.

![enter image description here](https://habrastorage.org/files/b76/124/ddf/b76124ddf46a4c66b856fa1940838e26.png)

Данное расширение появилось еще в марте 2016 года. И отзывы о нем не самые хорошие. Видимо поэтому почти целый год оно оставалось в тени.

Так что же такое Visual C++ for IoT Development? Оказалось что это "побочный" продукт попыток подключить [GDB Debugger](https://ru.wikipedia.org/wiki/GNU_Debugger) к Visual Studio. 

Эта история [началась](https://blogs.msdn.microsoft.com/visualstudioalm/2014/11/12/debugging-c-code-on-android-with-visual-studio-2015/) в 2014 году. Сначала в Visual Studio появилась возможность удаленной отладки С++ кода под Android. Через пол года была [анонсирована](https://blogs.msdn.microsoft.com/vcblog/2015/04/29/debug-c-code-on-linux-from-visual-studio/) возможность удаленной отладки С++ кода уже просто под Linux системами. В ноябре 2015 [появилась](https://blogs.msdn.microsoft.com/vcblog/2015/11/18/announcing-the-vs-gdb-debugger-extension/) Preview версия. Ну а сейчас дополнение для удаленной отладки кода на Linux из-под Visual Studio уже полностью [доступно](https://marketplace.visualstudio.com/items?itemName=VisualCPPTeam.VisualCforLinuxDevelopment) и активно развивается. Последнее обновление было 22.12.2016. Об этом уже даже [писали на хабре](https://habrahabr.ru/company/microsoft/blog/319962/).

Причем тут IoT? А оказалось, что уже давным-давно существует проект под названием [OpenOCD](http://openocd.org/). Полное его название Open On-Chip Debugger Free and Open On-Chip Debugging, In-System Programming and Boundary-Scan Testing. Смысл данного проекта заключается в том, что он является "единым" отладочным интерфейсом практически для любого микроконтроллера. Аппаратная ли система отладки или программная, внешний ли JTAG/SWD адаптер или он "размазан" на отладочной плате - ему все равно. На текущий момент OpenODC имеет конфигурационные файлы практически под все существующие отладочные средства. И унифицирует он все это как раз в GDB. 

Выглядит это примерно так:

![enter image description here](https://habrastorage.org/files/747/87f/4ed/74787f4edcb649c0bc2706c4d26579d3.png)

При этом открывается сокет по адресу localhost:3333. Соответственно это и есть GDB сервер, который позволяет выполнять удаленную отладку. К нему то и подключается Visual Studio.

В общем-то, это все. Больше Visual C++ for IoT Development ничего нового  делать толком и не умеет. Да, там можно настроить NMake и все-таки сделать компиляцию в консольном режиме. И еще есть настройки IntelliSense. Но это все и так было в проектах Makefile.

![enter image description here](https://habrastorage.org/files/aa5/1a4/4be/aa51a44be40742559203bb7a58d90473.png)

На текущий момент разработчики протестировали все совместно с [ARM GCC tools](https://launchpad.net/gcc-arm-embedded). [OpenOCD](http://openocd.org/), [pyOCD](https://github.com/mbedmicro/pyOCD) и JTAG отладчиком [Segger](https://www.segger.com/).

Чего бы хотелось от этого дополнения? Полноценной работы с ARM GCC так же как с обычными проектами, а не настраивать NMake. Тогда можно весь проект будет вести в Visual Studio, пользуясь всеми ее удобствами.

Причем технических проблем для этого нет. Мало того, такие попытки уже [делались](https://habrahabr.ru/post/251795/).  Даже есть коммерческие проекты - [VisualGDB](http://visualgdb.com/download/). Почему это не сделает сама Microsoft - остается загадкой.

Подводя итоги, можно сказать, что Visual C++ for IoT Development это здорово и пользоваться им вполне можно. Но неудобно. Есть несколько сценариев и все они предполагают "костыли" в лучшем случае в виде NMake. Надеюсь, что это расширение не будет заброшено и будет так же активно развиваться, как и [Visual C++ for Linux Development](https://habrahabr.ru/company/microsoft/blog/319962/). Хотя, так как последнее обновление было 31.03.2016, надежды могут быть и напрасны.

