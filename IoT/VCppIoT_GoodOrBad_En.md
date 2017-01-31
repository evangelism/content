

Visual C++ for IoT Development: Прорыв или разочарование?
=========================================================

![enter image description here](https://habrastorage.org/files/a26/acc/d5b/a26accd5b8fe481387bf227d97854eaa.png)

As you know world is changing very fast. Especially in IT and Software development. And the most difficult thing is to find information about all new features and take best of them. And as good example of this process I just found "new" Visual Studio Extencion called Visual C++ for IoT Development. Potentially it is very usefull thing but now it's name is better than it's content. Let's look it closer.


----------
The idea of using Visual Studio for microcontrollers firmware development visited me regulary for a long time. And now this idea transforms to practice. So all my projects closely integrated with Visual studio. In the begining I used it as a version control tool. Then I began to partially write code in it. And now I use integrated unit tests subsystem. But I always use external tools for compilation and debugging.

When I saw the header of Visual C++ for IoT Development I thought that this is the end of all my problems. I will stop to write code in one place and compile and debug in other place while switching between them many times. There was hope to do everything in Visual Studio.

![enter image description here](https://habrastorage.org/files/b76/124/ddf/b76124ddf46a4c66b856fa1940838e26.png)

This extension was published in March of 2016. But most of responses about is are not good. Maybe that was the reason of its unpopularity.

So what is the Visual C++ for IoT Development? It turned out that this is side product of using of [GDB Debugger](https://ru.wikipedia.org/wiki/GNU_Debugger) with Visual Studio.

This story [beginned](https://blogs.msdn.microsoft.com/visualstudioalm/2014/11/12/debugging-c-code-on-android-with-visual-studio-2015/) in 2014. First there was possibility of remote debugging C++ code for Android in Visual Studio. Support of remote debugging for Linux was [announced](https://blogs.msdn.microsoft.com/vcblog/2015/04/29/debug-c-code-on-linux-from-visual-studio/) next half of year. В ноябре 2015 [появилась](https://blogs.msdn.microsoft.com/vcblog/2015/11/18/announcing-the-vs-gdb-debugger-extension/) Preview версия. Ну а сейчас дополнение для удаленной отладки кода на Linux из-под Visual Studio уже полностью [доступно](https://marketplace.visualstudio.com/items?itemName=VisualCPPTeam.VisualCforLinuxDevelopment) и активно развивается. Последнее обновление было 22.12.2016. Об этом уже даже [писали на хабре](https://habrahabr.ru/company/microsoft/blog/319962/).

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


