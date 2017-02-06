Visual C++ for Linux Development: Практика использования для Windows разработчиков
========================================================================

![enter image description here](https://habrastorage.org/files/35a/daa/b6a/35adaab6a2ab44e59453a2002b6345ea.png)

Так получилось, что за достаточно долгу карьеру Windows и Embedded разработчика судьба свела меня по серьезному с Linux всего лишь несколько месяцев назад. Нужно было написать не очень сложную консольную программу. На тот момент все мои знания о Linux были взяты из курса по операционным системам в вузе (10 лет назад). Но Stackoverflow, google и опыт позволили достаточно быстро справиться с задачей. В итоге все было написано в Visual Studio Code под Ubuntu 14.04. Правда, приложение под Linux являлось только лишь небольшим клиентом для Windows сервера. Поэтому результат не очень удовлетворял меня, так как был оторван от основного проекта в Visual Studio. И только сейчас я смог перенести код в основной проект с помощью [Visual C++ for Linux Development](https://marketplace.visualstudio.com/items?itemName=VisualCPPTeam.VisualCforLinuxDevelopment). В процессе мне пришлось решить несколько сопутствующих проблем. Об этом я рассажу под катом.


----------

Итак, Visual C++ for Linux Development - это расширение для Visual Studio, позволяющее писать код в привычной многим IDE под Windows а отлаживать его прямо в целевой операционной среде - Linux. При этом используется GCC и Remote GDB Debugger. Более подробно о расширении можно прочитать в [блоге разработчиков](https://blogs.msdn.microsoft.com/vcblog/2016/03/30/visual-c-for-linux-development/) или в [переводе на хабре](https://habrahabr.ru/company/microsoft/blog/319962/).

Инструкции того, как установить, запустить, настроить и т.д. можно найти по ссылкам выше. У меня с этим не возникло никаких проблем. Вопросы начались со стороны Linux системы. Напомню, что я использую [Ubuntu 14.04 LTS](http://releases.ubuntu.com/14.04/) и дальнейшее изложение пойдет именно про нее. Если ком интересно, я использовал образ для [VirtualBox](https://www.virtualbox.org/) с сайта [osboxes.org](http://www.osboxes.org/ubuntu-14-04-5-trusty-images-available-virtualbox-vmware/). 

Так же, прошу сильно меня не ругать, я все-таки в Lunux далеко не гуру. Лучше подскажите, если что-то можно сделать более оптимальным путем.

Отладка первой программы
------------------------

Перед тем, как использовать удаленную отладку, нужно установить несколько компонентов на Linux системе. Как указано в инструкциях по ссылке выше, это можно сделать, выполнив в командной строке следующее:

    sudo apt-get install openssh-server g++ gdb gdbserver

Вызвать терминал в Ubuntu можно комбинацией клавиш *Ctrl+Alt+T*.

![enter image description here](https://habrastorage.org/files/e1f/d76/949/e1fd76949a994d74a16e4d98180778ff.png)

Я не помню, запускается ли все это хозяйство сразу или нет, по этому на всякий случай можно перезагрузиться.

Если все сделано правильно, то будет открыт порт 22. Проверить это можно используя команду `nmap`.

![enter image description here](https://habrastorage.org/files/d6b/65d/721/d6b65d7214f24bb5ac43ca86cf090fe4.png)
Но сразу подключиться из-под Visual Studio у меня не удалось, так как система почему то не пускала меня под единственным пользователем. Пришлось создать другого. Это можно сделать в *System Settings->User Account*.

![enter image description here](https://habrastorage.org/files/da1/fc0/99b/da1fc099bfd64161be9968ec55af2fc4.png)

При этом, не забыв нажать кнопку Unlock в верхнем правом углу.

![enter image description here](https://habrastorage.org/files/4bc/73d/370/4bc73d3701b041349a3f32c62303acef.png)

Настроить подключения в Visual Studio можно в окне *Tools->Options*

![enter image description here](https://habrastorage.org/files/646/b0b/f60/646b0bf60ee8426e9661078d9057ef87.png)

Теперь можно запустить и отладить тестовый проект.

![enter image description here](https://habrastorage.org/files/2ba/7fc/1b5/2ba7fc1b5ff94bcca10e3be543bf02a7.png)

При этом в Ubuntu будут скопированы исходники и собранный файл программы (если это не отключено в настройках проекта). Все это можно будет найти в папке */home/<имя пользователя>/projects*.

В моем случае получилось вот так:

![enter image description here](https://habrastorage.org/files/c29/440/39c/c2944039ca4442fc8c1c629f08464f02.png)

Запустить программу в самом Linux можно из консоли:

![enter image description here](https://habrastorage.org/files/c04/08f/8af/c0408f8afa5a4fe1b95894021da73687.png)

Теперь вроде бы можно начинать работать. Я перенес исходные файлы в Visual Studio и... ничего у меня не скомпилировалось. Оказалось, что проекту не достает *.h* файлов из *include directories*.

Подключаемые файлы
------------------

Вместе с Visual C++ for Linux Development устанавливается и множество заголовочных файлов. Их можно найти тут:


    C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\Linux\include\usr
  Но моему проекту этого не хватило. 

В [блоге разработчиков](https://blogs.msdn.microsoft.com/vcblog/2016/03/30/visual-c-for-linux-development/#tips) по этому поводу сказано следующее:

    Getting your include files
    Everyone loves IntelliSense, but we’re not yet synchronizing the include files from your Linux system. Everyone has their own ways to either share or copy these files which is great.

В будущем эту проблему обещают решить, ну а сейчас крутитесь как хотите. Там же приведен пример с копированием директории всей /usr/include с помощью [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

Но мне такой путь не нравится. Лично я предпочитаю расшарить папку с заголовочными файлами. Список директория для поиска include файлов можно посмотреть, выполнив в консоли команды

    $ touch a.c
    $ gcc -v -E a.c


![enter image description here](https://habrastorage.org/files/028/94e/6f6/02894e6f696040f68c0233628a592ff2.png)

Мне хватило папки `/usr/include`.

В случае с моей версией системы, перед тем как расшарить данную папку, нужно перевести ее во владение текущему пользователю. Делается это командой `sudo chown -R osboxes:test '/usr/include'`.

После этого можно открыть доступ к папке. Как это сделать написано [тут](http://ubuntuhandbook.org/index.php/2015/02/share-a-folder-in-ubuntu-14-04/).

После этого, сетевые пути можно прописать в Visual Studio как Include Directories.

Такой подход имеет преимущество в виде того, что вы будете работать всегда с оригинальными заголовочными файлами и вам не нужно будет ничего синхронизировать. С другой стороны, будут проблемы при переносе разработки на другой компьютер. Так же, как я уже писал, я работаю с Ubuntu, установленной на виртуальной машине на моем компьютере. При такой конфигурации, проблемы с безопасностью уходят на второй план.

Но в других конфигурациях, действия, описанные мной выше, могут быть запрещены.

Таком образом, проблему синхронизации заголовочных файлов нужно решать исходя из условий работы. Тут выбор остается за вами.

Дополнительные команды компилятора и линкера
-------------------

Заголовочные файлы стали видны и компиляция прошла успешно. Но вот слинковаться проекту не удалось. Дело в том, что я использую потоки и заголовочный файл "pthread.h". Для того, чтобы линкер увидел библиотеку pthread, нужно [использовать опцию](http://stackoverflow.com/questions/23250863/difference-between-pthread-and-lpthread-while-compiling) -pthread или -lpthread. 

Для этих целей в Visual C++ for Linux Development есть специальная настройка:
![enter image description here](https://habrastorage.org/files/a11/da3/5d0/a11da35d08144bbf8ba31b7d563da200.png)

Но у меня почему то она не работает. Проблема эта временная (разработчики уже знают об этом), но решать ее нужно здесь и сейчас. Обойти это ошибку можно используя другую опцию:

![enter image description here](https://habrastorage.org/files/0bd/91f/912/0bd91f91275c43eb8d3dfd9822200f84.png)

Если в место g++ написать g++ -pthread, то получится правильная строка линкера:
![enter image description here](https://habrastorage.org/files/b33/83b/e1d/b3383be1d2b64c8cadacc015b5b5109e.png)
![enter image description here](https://habrastorage.org/files/6e7/a0f/fa0/6e7a0ffa00fd4907a53054b43c753ea5.png)

Тот же трюк срабатывает и для компилятора.

Запуск отладки с правами администратора
---------------------------------------

Теперь все скомпилировалось и слинковалось. Однако для работы программы нужны повышенные права, так как она открывает файл устройства ввода. Соответственно, отладку так же нужно запускать с правами администратора. Сейчас в Visual C++ for Linux Development эта опция не реализована, но есть одно решение.

Можно повысить в правах gdb и gdbserver командами

    $ sudo chmod ugo+s /usr/bin/gdb
    $ sudo chmod ugo+s /usr/bin/gdbserver

Такой совет можно найти в комментариях в посту в [блоге разработчиков](https://blogs.msdn.microsoft.com/vcblog/2016/03/30/visual-c-for-linux-development/).

Этот трюк работает, но он не безопасен. По сути, вы отдаете свою систему любому, кто подключается к gdbservr. В моей конфигурации это не страшно, так как все запущено на одном моем компьютере, но в других условиях нужно быть очень аккуратными с такими действиями.

Копирование дополнительных файлов
---------------------------------

И остался последний момент. Моя программа читает настройки из текстового файла. Он является частью проекта Visual Studio и при компиляции должен копироваться в папку с исполняемым файлом.

Это можно так же сделать в настройках проекта:
![enter image description here](https://habrastorage.org/files/577/141/63e/57714163eaef4e06af821d51589f3f2d.png)

Чтобы просто скопировать файл, как остальные исходники, его можно добавить в поле Sources To Copy: @(SourcesToCopyRemotely);config.txt

А скопировать его в другую директорию можно с помощью Additional Sources To Copy.

Формат этой настройки
`fulllocationpath1:=fullremotepath1;fulllocationpath2:=fullremotepath2` 
и т.д.

В моем случае такая строка выглядит так:

    $(ProjectDir)config.txt:=$(RemoteOutDir)config.txt;

Все бы хорошо, но и тут у меня возникли проблемы.
Дело в том, что макрос $(RemoteOutDir) раскрывается в путь, начинающийся с символа "~".

Судя по всему, этот путь завязан с этой настройкой:

![enter image description here](https://habrastorage.org/files/601/129/787/6011297871804e16bbf1967740e445e6.png)

Так вот, при компиляции все работает хорошо. Но вот при копировании файлов почему то ~ воспринимается не как root директория, а просто как имя папки. То есть создается папка с именем "~":
![enter image description here](https://habrastorage.org/files/626/11b/a77/62611ba77d584c43b7e95ff3037aef7b.png)

Справиться с этим мне так и не удалось, поэтому я просто копировал файл config.txt вручную. Правда, для этого пришлось опять использовать изменение прав на папку:
sudo chown -R osboxes:test '/home/test'


Что в итоге
-----

Лично я могу сказать, что Visual C++ for Linux Development extension мне помог. Не смотря на все проблемы и пару багов, он позволил мне быстрее и эффективнее решить задачу, связанную с разработкой под Linux. 

Наверное, можно на это возразить, что есть более удобные пути, но я исходил только из своего опыта и знаний, а все это в основном связано с Windows.


p.s. Недавно в сети появилось [видео](https://www.youtube.com/watch?v=BVVIioKfJG8) от MSP Александра Поповкина, где он так же делает обзор данного дополнения. 
