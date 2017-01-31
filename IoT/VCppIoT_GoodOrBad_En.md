

Visual C++ for IoT Development: Breakthrough or disappointment?
=========================================================

![enter image description here](https://habrastorage.org/files/a26/acc/d5b/a26accd5b8fe481387bf227d97854eaa.png)

As you know world is changing very fast. Especially in IT and Software development. And the most difficult thing is to find information about all new features and take best of them. And as good example of this process I just found "new" Visual Studio Extencion called Visual C++ for IoT Development. Potentially it is very usefull thing but now it's name is better than it's content. Let's look it closer.


----------
The idea of using Visual Studio for microcontrollers firmware development visited me regulary for a long time. And now this idea transforms to practice. So all my projects closely integrated with Visual studio. In the begining I used it as a version control tool. Then I began to partially write code in it. And now I use integrated unit tests subsystem. But I always use external tools for compilation and debugging.

When I saw the header of Visual C++ for IoT Development I thought that this is the end of all my problems. I will stop to write code in one place and compile and debug in other place while switching between them many times. There was hope to do everything in Visual Studio.

![enter image description here](https://habrastorage.org/files/b76/124/ddf/b76124ddf46a4c66b856fa1940838e26.png)

This extension was published in March of 2016. But most of responses about is are not good. Maybe that was the reason of its unpopularity.

So what is the Visual C++ for IoT Development? It turned out that this is side product of using of [GDB Debugger](https://ru.wikipedia.org/wiki/GNU_Debugger) with Visual Studio.

This story [began](https://blogs.msdn.microsoft.com/visualstudioalm/2014/11/12/debugging-c-code-on-android-with-visual-studio-2015/) in 2014. First there was possibility of remote debugging C++ code for Android in Visual Studio. Support of remote debugging for Linux was [announced](https://blogs.msdn.microsoft.com/vcblog/2015/04/29/debug-c-code-on-linux-from-visual-studio/) next half of year. Preview version [appeared](https://blogs.msdn.microsoft.com/vcblog/2015/11/18/announcing-the-vs-gdb-debugger-extension/) in November of 2015. And now Visual C++ for Linux Development [can be downloaded](https://marketplace.visualstudio.com/items?itemName=VisualCPPTeam.VisualCforLinuxDevelopment) as version 1.0.6. Last update was from 12.22.2016.

And where is IoT? It turns out that there is a project called [OpenOCD](http://openocd.org/). It full name is Open On-Chip Debugger Free and Open On-Chip Debugging, In-System Programming and Boundary-Scan Testing.  The main idea of this project is that implements one central debug interface endpoint for most of microcontrollers. Never mind if it software or hardware debug system, external JTAG/SWD device or integrated debug module. At this moment OpenOCD has config files for most popular debug systems. And as debug standard it uses same GDB.

It looks like this:

![enter image description here](https://habrastorage.org/files/747/87f/4ed/74787f4edcb649c0bc2706c4d26579d3.png)

Debugger opens socket at localhost:3333. This endpoint is GDB server and Visual Studio using it as remote debugger server.

And this is all. Visual C++ for IoT Development can't do anything new.
Yes, you cant configure NMake and use compilation in console mode. Also you can configure IntelliSense. But Makefile projects already contains these features.


![enter image description here](https://habrastorage.org/files/aa5/1a4/4be/aa51a44be40742559203bb7a58d90473.png)

At this point the extension was tested with  [ARM GCC tools](https://launchpad.net/gcc-arm-embedded). [OpenOCD](http://openocd.org/), [pyOCD](https://github.com/mbedmicro/pyOCD) и [Segger](https://www.segger.com/) JTAG debugger.

What can we want from it ? Full support of ARM GCC like other standard project types without using NMake. Then it will be possible to use Visual Studio and it's cool features in all development and debugging process.

And there is no technical problems for it. Moreover, such attempts already was made. You even can find commercial projects. For example [Visual GDB](http://visualgdb.com/download/). Why Microsoft still did not do it - one big question.

Summing up, we can say, that Visual C++ for IoT Development is good and you really can use it. But it is uncomfortable. There is some scenarios of it's using and all of them includes additional difficulties such using NMake. I hope this extension will progress as fast as Linux remote debugging. But last update was on 03.31.2016 so my hope can be in vain...
