Новая инициатива Microsoft OpenT2T: "Интерфейс всего"

![enter image description here](https://habrastorage.org/files/9cb/8a0/3de/9cb8a03de47d4c7884c079473b4e2644.png)

Microsoft уже давно проявляет заметную активность в сфере Интернета вещей: Специализированные облачные сервисы,  IoT редакция Windows 10, версия .Net для микроконтроллеров, множество хакатонов и т.д. Не так давно [был представлен](https://blogs.windows.com/buildingapps/2016/04/05/open-translators-to-things-an-open-approach-for-accessing-similar-things/) новый проект, который, теоретически, может существенно упростить жизнь многим разработкам IoT систем. Называется он Open Translators to Things(OpenT2T). Если двумя словами, то этот проект можно описать как "Интерфейс всего". Ну а более подробное описание - далее в этой статье.


----------


Любой, кто разрабатывал программное обеспечение для устройств, сталкивался с большим разнообразием протоколов и подходов при работе с одним и тем же классом компонентов. Казалось бы, все датчики температуры работают примерно одинаково. Нужно просто считать с них показания и все. Однако, на практике происходит следующее:

Чтение данных с температурного датчика [LM35](http://www.ti.com/lit/ds/symlink/lm35.pdf) происходит так (взято [отсюда](http://duino4projects.com/arduino-temperature-sensor-code/)):

    tempCC = (5.0 * reading(tempPin) * 100.0) / 1024.0 
	
Для датчика [TMP36](http://www.analog.com/media/en/technical-documentation/data-sheets/TMP35_36_37.pdf) код будет такой (взято [отсюда](https://learn.adafruit.com/tmp36-temperature-sensor/using-a-temp-sensor)):

    tempC = (5.0 * reading(tempPin) -  0.5) * 100

А если использовать термистор в 10K Ом, то может получиться такой код (взято [отсюда](http://computers.tutsplus.com/tutorials/how-to-read-temperatures-with-arduino--mac-53714)):

    tempC = log(((10240000/ reading(tempPin)) - 10000))
    tempC = 1 / (0.001129148 + (0.000234125 + 0.0000000876741 * tempC ^ 2)) * tempC )
    tempC = temp - 273.15

А вот, например, разные варианты работы с лампочкой:

[Philips Hue](http://www.philips.ru/c-p/8718291241737/hue-personal-wireless-lighting) (взято [отсюда](http://www.developers.meethue.com/documentation/core-concepts)):

    PUT http://<bridge ip>/api/<username>/lights/1/state {"bri":254}

[Wink Hub](http://www.wink.com/products/wink-hub/) (взято [отсюда](http://docs.wink.apiary.io/#reference/device/light-bulb)):


    PUT https://api.wink.com/light_bulb/<device_id>  {“desired_state": { “powered” : true} }

И так во всем.

В Microsoft решили попробовать побороть данную проблему с помощью новой инициативы -  [Open Translators to Things(OpenT2T)](http://www.opentranslatorstothings.org/#/). Идея заключается в том, чтобы попросить самих разработчиков помочь создать "схемы работы" для разных классов устройств. Нужно сказать, что эта идея не нова. Например, [существует](https://www.iab.org/wp-content/IAB-uploads/2016/03/OCF-oneIoTa-Overview-Paper_v3.pdf) Open Connectivity Foundation. Microsoft хочет объединить усилия всех таких групп, отдельных производителей и разработчиков. Специально для этого был создан [репозиторий на GitHub](https://github.com/openT2T), благодаря которому любой желающий может присоединиться к этому процессу.

Технически, "интерфейсы всего" предлагается создавать на  JavaScript/Node.js. Результат получается кросс-платформенным. Есть примеры для Windows, OS X, Android, Azure и AWS. Каждый "интерфейс" состоит из трех частей:

 1. Схема
 2. Транслятор
 3. Приложение

**Схема** объединяет устройства одного и того же класса. Это и есть обобщенный "интерфейс" для одинаковых устройств. Для примера на GitHub [есть схема](https://github.com/openT2T/translators/blob/master/org.OpenT2T.Sample.SuperPopular.TemperatureSensor/org.OpenT2T.Sample.SuperPopular.TemperatureSensor.xml) для датчиков температуры. Сейчас используются схемы в [формате AllSeen](https://wiki.allseenalliance.org/irb/extended_introspection_xml).

**Транслятор** - это реализация схемы для конкретного устройства. Он должен скрывать особенности работы с устройством, что должно позволить разработчикам не тратить время на изучение документации, а с разу преступить к работе. Трансляторы могут быть запущены как на локальных устройствах (ПК, телефоны, планшеты, специализированные контроллеры и т.д.), так и в облаке. Все зависит от принципов работы управляемого устройства. [Примеры трансляторов](https://github.com/openT2T/translators) написаны на JavaScript с использованием Node.js. Но никто не запрещает использовать и другие языки и платформы - C++, C#, Java и т.д.

**Приложение** - это примеры использования схемы и транслятора для разных платформ.

Как же этим все пользоваться? Предположим, мы хотим считать температуру с датчика [CC2650](http://www.ti.com/product/CC2650).

 1. Для начала, нам нужно найти подходящую схему. В нашем случае она находится [тут]( https://github.com/openT2T/translators/blob/master/org.OpenT2T.Sample.SuperPopular.TemperatureSensor/org.OpenT2T.Sample.SuperPopular.TemperatureSensor.xml). Если схема вас чем-то не устраивает, то можно вынести это на обсуждение и внести в нее правки.
 2. Затем найти и использовать подходящий вам транслятор или написать собственный. В нашем случае транслятор находится [тут](https://github.com/openT2T/translators/blob/master/org.OpenT2T.Sample.SuperPopular.TemperatureSensor/Texas%20Instruments%20SensorTag/js/thingTranslator.js).
 3. Далее написать приложение, использующее выбранное вами устройство. Для датчика CC2650 OpenT2T уже содержит несколько [тестовых](https://github.com/openT2T/sampleapps) примеров для разных платформ: [UWP С#](https://github.com/openT2T/sampleapps/tree/master/uwp), [Java для Android](https://github.com/openT2T/sampleapps/tree/master/android), [Azure IoT Hub](https://github.com/openT2T/sampleapps/tree/master/azure) и [AWS Kinesis](https://github.com/openT2T/sampleapps/tree/master/aws).


Чтобы понять какую помощь ожидает Microsoft от сообщества нужно заглянуть в [Roadmap](http://www.opentranslatorstothings.org/#/roadmap). Проект сейчас находится в Alpha стадии и разделен на несколько частей:

 - **Схемы и трансляторы.** Всем желающим предлагается поучаствовать в формировании схем устройств и создании новых трансляторов.
 - **Console.** Предполагается, что разработчики будут использовать консольные приложения на своих устройствах для первоначальной настройки модулей. Консольное приложение содержит node.js процесс, который распознаёт новые устройства и загружает соответствующий транслятор. Тут требуется помощь для создания полноценных приложений для всех операционных систем (Windows, OS X, Android и iOS).
 - **Облако.** Работа с устройствами может осуществляться и через разнообразные облака. На этом направлении много работы, связанной с поддержкой разных протоколов и форматов для пересылки данных в облачные сервисы.
 - **CLI.** Сommand Line Interface , который будет использоваться разработчиками для тестирования JavaScript трансляторов. От сообщества ожидают помощи для доработки этого инструмента.
 - **Sampleapps.** Репозиторий уже содержит примеры для разных платформ. Но, чем больше примеров, тем лучше. По этому, предлагается добавлять примеры для всего, что вам интересно. Например, актуальными будут Xamarin, Cordova или Swift, а так же другие облака (Google Cloud).
 - **Voice.** Добавление новый голосовых файлов позволит использовать Cortana и других голосовых помощников для управления устройствами.

Принять участие в этом процессе достаточно просто. Можно начать с просмотра [этого](https://channel9.msdn.com/series/opent2t/Open-Translators-to-Things-Introduction-for-Users) и [этого](https://channel9.msdn.com/series/opent2t/Getting-Started-with-Open-Translators-to-Things) видео. Можно почитать [tutorial](http://www.opentranslatorstothings.org/#/gettingStarted). А можно поучаствовать в [дискуссии](https://github.com/openT2T/opent2t.github.io/issues) на GitHub. Другие площадки для общения находятся [тут](http://openconnectivity.org/) и [тут](https://www.iab.org/activities/workshops/iotsi/). Приглашаются все желающие.

Подводя итог, можно отметить, что идея правильная и своевременная. Но, на мой взгляд, несколько наивная. Вызывает вопросы и [стек технологий](https://github.com/openT2T/opent2t.github.io/issues/3). Но, учитывая ресурсы, которые Microsoft может включить в этот проект, из него может что-то и получиться. В любом случае, инициатива полезна, как площадка для обсуждения проблемы и поиска ее решения. А это уже половина дела.
   
