![enter image description here](https://habrastorage.org/files/964/b80/438/964b804382664f4a81a7a1f7f948f33e.jpg)

Существует огромное количество примеров и статей про Windows 10 IoT Core, рассказывающих о том, как легко и удобно делать с его помощью разнообразные устройства. Однако в реальности работа с любым "железом" всегда связана со множеством не самых очевидных нюансов, знание которых приходит только с практикой. Я расскажу о некоторых особенностях работы c GPIO на Raspberry Pi2 и Windows 10 IoT Core и  заодно о новой функции Remote Client, доступной в версии Insider Preview.


----------

Началось все с того, что мне нужно было получить номер карты со считывателя системы СКУД (контроля доступа). Почти все считыватели умеют передавать эти данные по интерфейсу [Wiegand](https://ru.wikipedia.org/wiki/%D0%92%D0%B8%D0%B3%D0%B0%D0%BD%D0%B4_%28%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D1%84%D0%B5%D0%B9%D1%81%29). Он представляет собой 3 провода: сигнальный для передачи единиц, сигнальный для передачи нулей и земля. В режиме ожидания на каждом сигнальном проводе устанавливается 5В. Данные передаются "обратными" импульсами. Ширина импульсов от 50 до 200 мкс, период от 300 до 3000 мкс:

![enter image description here](https://habrastorage.org/files/b34/023/8db/b340238dbc784cb9a24a7201c8a8bdea.PNG)

Данные идут всегда в одну сторону от считывателя к контроллеру. Количество бит может варьироваться и их интерпретация тоже может быть разной. Окончание посылки определяется тайм-аутом от 50 до 250 мс.

Такой разброс параметров произошел потому, что это "исторически сложившийся" интерфейс, который не имеет четкого стандарта.

Мне досталось устройство с протоколом wiegand26 - посылка в нем содержит 26 бит, в которых 2 бита контроля четности.

Задание было дано в рамках создания демонстрационного стенда, так что можно было поэкспериментировать с платформой. Поэтому очень кстати пришлась Raspbery Pi2 с Windows 10 IoT Core на борту.

Проблемы с GPIO
---------------

Решить эту задачу можно двумя способами:

 - использовать прерывания при изменении напряжения на пинах
 - использовать опрос пинов

Самым простым мне показался первый вариант. Правильно инициализировав пины, можно не грузить процессор постоянным опросом, а отрабатывать только асинхронные вызовы. Код тут очень прост:

    var gpio = GpioController.GetDefault();
    
    var data0 = gpio.OpenPin(data0Pin);
    var data1 = gpio.OpenPin(data1Pin);
        
    // Check if input pull-down resistors are supported
    if (data0.IsDriveModeSupported(GpioPinDriveMode.InputPullUp))
    data0.SetDriveMode(GpioPinDriveMode.InputPullUp);
    else
    data0.SetDriveMode(GpioPinDriveMode.Input);
    
    if (data1.IsDriveModeSupported(GpioPinDriveMode.InputPullUp))
    data1.SetDriveMode(GpioPinDriveMode.InputPullUp);
    else
    data1.SetDriveMode(GpioPinDriveMode.Input);
    
    var ticksPerMillisecond = TimeSpan.TicksPerMillisecond;
    var tenMs = ticksPerMillisecond / 1000; //1 микросекунда
    
    data0.DebounceTimeout = TimeSpan.FromTicks(tenMs);
    data0.ValueChanged += data0_ValueChanged;
    data1.DebounceTimeout = TimeSpan.FromTicks(tenMs);
    data1.ValueChanged += data1_ValueChanged;

Сразу нужно сказать про DebounceTimeout. Вообще он тут не нужен, так как, как правило, при нормальной работе интерфейса без существенных наводок "дребезга" в нем нет. Но из-за того, что в общем случае мы не знаем, с каким считывателем и в каких условиях будет все это работать, то я решил поэкспериментировать с этим таймаутом.

Но ничего хорошего из этого не получилось. В примерах DebounceTimeout везде задается в миллисекундах, но для моего случая нужны микросекунды. Какое бы значение я не устанавливал, события ValueChanged не появлялись. Поэтому таймаут пришлось просто отключить, и все стало работать нормально:

	data0.DebounceTimeout = TimeSpan.FromTicks(0);
	data0.ValueChanged += data0_ValueChanged;
	data1.DebounceTimeout = TimeSpan.FromTicks(0);
	data1.ValueChanged += data1_ValueChanged;

Данные считывались так:

	private void data0_ValueChanged(GpioPin sender, pioPinValueChangedEventArgs e)
	   {
            if (e.Edge == GpioPinEdge.FallingEdge)
            {
                UpdateValue(0);
            }
        }
	private void data1_ValueChanged(GpioPin sender, pioPinValueChangedEventArgs e)
	   {
            if (e.Edge == GpioPinEdge.FallingEdge)
            {
                UpdateValue(1);
            }
        }

Затем пришла пора решать вопрос с моментом окончания посылки. Как я писал выше, окончанием посылки считается отсутствие импульсов на обоих сигнальных контактах в течение минимум 50 мс. Тут отлично подошел бы таймер, который бы регулярно проверял, был ли обмен, и если нет, то отправлял бы полученный результат дальше. 

Такой подход работал плохо. Я использовал ThreadPoolTimer, так как код должен был работать и без UI. Особенностью этого таймера является то, что каждый раз его обработчик вызывается в новом потоке. Если обработчик выполняется дольше периода таймера, то параллельно будет запущен еще один, а потом еще один, и так далее. При этом в нашем случае, в обработчике таймера должны быть сброшены данные, чтобы начать новый прием. Поэтому он является критической секцией и не должен выполняться одновременно более чем в одном экземпляре. Соответственно, короткие интервалы таймера использовать было нельзя.
Далее оказалось, что и длинные интервалы тоже нельзя использовать, как и синхронизацию обработчика через lock, так как пока выполняется обработчик таймера, перестают приходить события ValueChanged от портов. Похоже, что эти события менее приоритетны, чем обработчики таймеров. Причем эксперименты показали, что события не висят в очереди, а просто молча отбрасываются.

Поэтому, как ни крути, использование таймера приводило к потере событий от GPIO и, как следствие, потере данных. Их можно было уменьшить путем подбора периода таймера и перераспределения вычислений на ValueChanged. Но абсолютной надежности достичь с таким подходом нельзя даже теоретически. Кроме того, выяснилось, что к пропуску событий и потере данных ведет и слишком долгое выполнение ValueChanged.

Я решил использовать вместо таймера поток. Для синхронизации применялась queue, в которую складывались пришедшие в ValueChanged биты. Поток должен некоторое время спать, затем проверять, был ли обмен за это время, и если не было, то забирать биты из очереди, собирать из них результат и отправлять дальше.

Но и тут ничего не вышло. Можно было предположить, что ValueChanged не будет вызываться при работе потока, но вот то, что он не вызывается, когда поток спит, для меня оказалось сюрпризом. В качестве аналога Thread.Sleep я использовал Task.Delay. Не знаю, происходит ли при этом выполнение других Task'ов (судя по документации, Delay запускает еще один Task с таймером), но события от GPIO это все наглухо блокирует.

В общем, драйвер GPIO для Windows 10 IoT написан так, что его почти невозможно использовать асинхронно, так как его события имеют кране низкий приоритет в системе.

Lightning
=========

В процессе изучения работы с GPIO я узнал, что существует еще один, более быстрый драйвер, который можно использовать на Windows 10 IoT Core. Называется он [Lightning](https://ms-iot.github.io/content/en-US/win10/LightningProviders.htm) и включает в себя не только GPIO, но и работу с ADC, I2C, PWM, SPI. Быстрота работы достигается за счет "прямого доступа к памяти"(direct memory access).

Драйвер находится на стадии preview, но уже включен в версии Windows 10 IoT Core Insider Preview. Использовать его стоит тем, кому не хватает скорости работы стандартного драйвера. Я же надеялся тут получить более приоритетный ValueChanged.

По ссылке выше есть инструкция, как его использовать, НО, просто так он не заработает. Проблема заключается в NuGet пакете. Все устанавливается, но до namespace Microsoft.IoT.Lightning.Providers достучаться невозможно. Причем единственное упоминание о том, что проблему нельзя побороть, я нашел в [этой статье](http://blog.infernored.com/how-to-fade-an-led-with-pwm-in-windows-iot). Автор рассказывает как управлять светодиодами. В частности, он столкнулся с медленной работой PWM и исправил ситуацию с помощью использования Lightning.
Выглядит оно так:

	Third, you'll need to reference the Lightning SDK.  According to the documentation, you just reference via NuGet.  Unfortunately, this doesn't work as of v1.0.3-alpha.  I had to download the Microsoft.IoT.Lightning.Providers C++ source, add the Microsoft.Iot.Lightning.Providers.vcxproj project to my solution, and then make a project reference.  
	
	Incidentally, I contacted some folks at Microsoft, and they said a new nuget will be published shortly with binaries that will fix this issue.

Мне помог способ, описанный в статье. Я тоже скачал исходники, добавил в Solution файл проекта и сделал на него Reference. И какого же было мое удивление, когда оказалось, что события ValueChanged в Lightning не реализованы...

После этого я бросил идею использовать асинхронный подход и решил сделать опрос пинов в цикле. 

**UPDATE:** В процессе решения задачи я задал разработчикам [вопрос](https://github.com/ms-iot/BusProviders/issues/26) по поводу ValueChanged. Они ответили, что скоро все сделают. И свое обещание они [выполнили](https://github.com/ms-iot/BusProviders/commit/1e1e4eb74c9d1bc7e150082972682d47987cfdd5).

Кроме того ,в NuGet [появилась](https://www.nuget.org/packages/Microsoft.IoT.Lightning.Providers/) Microsoft.IoT.Lightning.Providers 1.0.0, которая делает видимым namespace Microsoft.IoT.Lightning.Providers.

Решение задачи
==============

В итоге задачу удалось решить путем создания цикла с периодом опроса пинов в 10 мкс. Сначала были опасения, что может не хватить скорости, но оказалось, что все работает достаточно быстро даже на стандартном драйвере.

Код выглядит примерно так:
	
	_data0 = gpio.OpenPin(data0Pin, GpioSharingMode.Exclusive);
	_data1 = gpio.OpenPin(data1Pin, GpioSharingMode.Exclusive);
	
	if (_data0.IsDriveModeSupported(GpioPinDriveMode.InputPullUp))
	    _data0.SetDriveMode(GpioPinDriveMode.InputPullUp);
	else
	    _data0.SetDriveMode(GpioPinDriveMode.Input);
	
	if (_data1.IsDriveModeSupported(GpioPinDriveMode.InputPullUp))
	    _data1.SetDriveMode(GpioPinDriveMode.InputPullUp);
	else
	    _data1.SetDriveMode(GpioPinDriveMode.Input);
	
	
	_task = Task.Run(() => TaskHandler());


Чтобы не блокировать выполнение других потоков, цикл сделан внутри Task:

	 private void TaskHandler()
	 {
	     var ticksPerMillisecond = TimeSpan.TicksPerMillisecond;
	     var mks = ticksPerMillisecond / 1000; //1 микросекунда
	     while (!_stopTask)
	     {
		     Task.Delay(TimeSpan.FromTicks(mks*10)).Wait();
			 var dt0 = _data0.Read();
	         var dt1 = _data1.Read();
	         .....
	      }
	}

Работает стабильно, но при условии, что карточки к ридеру подносятся не чаще чем 1 раз в секунду.

![enter image description here](https://habrastorage.org/files/317/224/4b9/3172244b9b3446ceb2bf6d9113856d05.jpg)

Данный проект со всеми инструкциями о подключении ридера к Raspberry Pi2 я планирую через пару недель выложить на GitHub.

Remote client
=============

Многие разработчики, пользующиеся Windows 10 IoT Core, отмечают, что им очень не хватает удаленного рабочего стола. Монитор или телевизор не всегда есть под рукой, да и пользоваться ими не всегда удобно. Наконец этот пробел был закрыт и, начиная с версии 10.0.14295.1000, [появился](https://developer.microsoft.com/en-us/windows/iot/win10/remotedisplay) удаленный клиент.  Сейчас эта и более новая версия доступны как [Insider Preview](https://ms-iot.github.io/content/en-US/win10/GetStarted/SetUpYourDeviceManually.htm).
Работает все очень просто. На стороне Windows 10 IoT Core нужно разрешить подключение удаленного клиента в веб интерфейсе:
![enter image description here](https://habrastorage.org/files/d34/f48/307/d34f483072bb4027bf330261ad13d240.png)

[Установить](https://www.microsoft.com/en-us/store/apps/iot-remote-client/9nblggh5mnxz) сам клиент
![enter image description here](https://habrastorage.org/files/508/e5e/dbb/508e5edbb30a4b2b973e2ebcc72066e6.png)

Запустить его и подключиться с Windows IoT Core
![enter image description here](https://habrastorage.org/files/22b/4c1/589/22b4c158929443eeae29a698d6d543c0.png)

В инструкции указано, что все хорошо работает на Raspberry Pi 2 и 3, Minnowboard Max и Dragonboard. При этом на Pi2 производительность немного ниже, так как там отсутствует поддержка GPU.

У меня как раз была Rapberry Pi2.
![enter image description here](https://habrastorage.org/files/4ac/11b/dfa/4ac11bdfae824166b3cf4c9df27e0218.jpg)

Небольшие задержки есть, но, на мой взгляд, они не критичны. При этом работает все стабильно и без проблем.


Выводы
======

Подводя итог всему вышесказанному, отмечу, что Windows 10 IoT Core работает не всегда так, как может ожидать от нее человек, привыкший к микроконтроллерам. Все-таки это полноценная операционная система, которая достаточно сильно абстрагируется от "железа":

 1. Приоритет событий от GPIO крайне низкий. Да и механизм событий не совсем понятен.
 2. Сложности с асинхронным программированием, непосредственно не связанные с работой прерываний.
 3. Существующего функционала не всегда хватает, а тот, что представлен в виде preview, не всегда корректно работает. Хотя развивается все это очень быстро.
 4. Производительности Windows 10 IoT Core вполне хватает для задач, чувствительных ко времени выполнения. При этом есть еще запас в виде Lightning.
 5. Появился Remote Client, что существенно повысило удобство работы.
