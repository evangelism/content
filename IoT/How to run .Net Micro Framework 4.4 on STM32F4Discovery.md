Introduction
------------

.Net Micro framework is a technology that allows to write applications for microcontrollers using the power of managed code and Visual Studio. It was founded long time ago and now it is big open-source project which has recently moved to GitHub. However, it is not "boxed" product yet. Work with .Net micro framework requires certain skills. In the last [article](http://www.codeproject.com/Articles/1079603/Getting-Started-with-the-NET-Micro-Framework). I wrote about how to create and run a simple "Hello world" application on the emulator for Windows. Now I will focus on how to work with the .Net micro framework on STM32F4Discovery board. Russian version if this article is [here](https://geektimes.ru/post/266612/).

Background
----------

The board is quite common and can be purchased for example [here](http://www.amazon.com/STM32F4DISCOVERY-STM32F407-Discovery-Evaluation-Development/dp/B00CW9AKDY). Starting with  the version 4.4 the port for this board is included in the netmf-interpreter [repository](https://github.com/NETMF/netmf-interpreter). Previously, it existed as a [separate project](http://netmf4stm32.codeplex.com/).

You can find some [materials](https://www.google.ru/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=STM32F4DISCOVERY%20netmf) about the launch of .Net micro framework on STM32F4Discovery  board, but they refer to version 4.3, and they use compiled assemblies. I'll talk about how to compile and run .Net micro framework version 4.4 on STM32F4Discovery board in detail. The article will be long, since it is necessary to fix a few issues in the distribution and download and install several utilities and drivers.

Preparing for compilation
-------------------------

First of all, you need to have that distribution.

You can find the repository [here](https://github.com/NETMF/netmf-interpreter). It can be downloaded as a zip-archive or can be obtained using git. Instructions on how to get the repository and build the installation files can be found [here](https://github.com/NETMF/netmf-interpreter/wiki/Getting%20Started). This article is based on these instructions. Versions of the repository associated with specific releases can be downloaded as a zip-archive from [here](https://github.com/NETMF/netmf-interpreter/releases).

Use these steps to get repository using git:

 1. Create public copy of the repository in your GitHub account on the GitHub servers with fork command. All pull requests must come from a public GitHub repository. 
 2. Get a local copy of the repository by using clone command. For example:

  `git clone https://github.com/<your GitHub ID>/netmf-interpreter.git`  

	**Important:** When you select a path to a local repository, it is necessary to make at least one parent folder. For example, `D:\NETMF\repo`, where the repo - folder for the repository.

 3. Set up a local repository as the Upstream. Establishing the official repository as the "Upstream" repository for your local clone enables you to pull changes from the latest official commits into your repository and resolve merge issues locally before submitting a pull request. The command for establishing the Upstream remote is:
`git remote add upstream https://github.com/NETMF/netmf-interpreter.git`  

**Important:** When you unpack the repository from a zip-file, you also need to make at least one parent folder. For example, `D:\NETMF\repo`, where the repo - folder for the repository.

Unfortunately .NET Micro Framework v4.4 Release To Web (RTW) contains issues that do not allow you to immediately build the installation files from the repository. However, these issues can be easily fixed.

When you get your own local copy of the repository, you need to do this steps:

 1. Download [binary tools](http://netmf.github.io/downloads/build-tools.zip) zip file. This file contains the tools needed to build both the installation files and "ports" for devices. In the future, it is planned to remove these utilities, but they are still needed.
 2. Unzip the contents of *binary tools* zip-file in the parent folder of the repository. For example, for the path `D:\NETMF\repo`, where the *repo* - the repository folder, the *bin* and *tools* folder must be in the folder `D:\NETMF`.

 3. **Important:** In the file 
 `<repo_folder>\Framework\Tools\BuildTasksInternal\BuildSigner\BuildSignerSpotBuild.csproj` 
 in line 37 change
 `<HintPath>$(MSBuildProgramFiles32)\Microsoft Internal\Codesign.Submitter\CODESIGN.Submitter.dll</HintPath>` 
 to
 `<HintPath>$(SPOROOT)\tools\x86\CODESIGN\CODESIGN.Submitter.dll</HintPath>`

 This is the first issue fix. Without such replacement, build of the repository will not succeed. This fix is needed only for .NET Micro Framework v4.4 Release To Web (RTW). In the future, the repository will already contain correct file. You can read [here](https://github.com/NETMF/netmf-interpreter/issues/333) about this issue.

 4. You need to download the CMSIS library and put it in the folder `<repo folder>\CMSIS`. More information about version and links to the CMSIS library can be found in the file `<repo folder>\CMSIS\ReadMe.md`.

 [CMSIS](http://www.arm.com/products/processors/cortex-m/cortex-microcontroller-software-interface-standard.php) stands for Cortex Microcontroller Software Interface Standard. This library does not depend on a particular manufacturer and intend to work with the Cortex-M core. CMSIS is supplied and supported by kernel developers - the ARM company. Using this library allows you to simplify the creation of the "ports" for different microcontrollers from different manufacturers.

 In case of version of .Net Micro Framework 4.4 you need to download CMSIS version 4.3 or newer. The library is supplied as a zip-archive (CMSIS-SP-00300-r4p3-00rel0.zip). It can be downloaded at ARM [web site](http://www.arm.com/products/processors/cortex-m/cortex-microcontroller-software-interface-standard.php). The content of the archive should be put in the folder `<repo folder>\СMSIS`.

 5. In this step you need to install the .Net Micro Framework Cryptographic Libraries. These libraries are used to sign the assemblies that will be executed on the microcontroller. Only executable files of cryptographic library are needed for build process. But you can download source code too.
 
 Libraries are available as msi file. I recommend to install them in any convenient folder (we will call it `<crypto install folder>`), and then copy them to the root of each repository. For example `D:\NETMF\repo` and `D:\NETMF\repo_master`.

The repository is a complex structure with lots of cross-references. It is combined by an [MSBuild](https://ru.wikipedia.org/wiki/MSBuild) project. Project files look like sln and proj files for Visual Studio but they have more complex structure inside. That's why Visual Studio can’t be used for building.

I'll discuss the components and references in the repository in future articles. But for now you need to know that the STM3F4Discovery port is located in the

   `<repo folder>\Solutions\STM32F4DISCOVERY`

and binary and hex files will be found in folder

`<repo folder>\BuildOutput`

MSBuild is a part of Visual Studio. The [documentation](https://github.com/NETMF/netmf-interpreter/wiki/Getting%20Started) for .netmf interpreter 4.4 states that the version of Visual Studio 2015 Community, Pro and Ultimate are supported, so you need to install one of them.

Next you need a compiler for ARM..Netmf interpreter 4.4 is supporting two compilers:

 - [GCC ARM Embedded](https://launchpad.net/gcc-arm-embedded) 
 - [ARM RealView Compilation tools](http://www.keil.com/arm/realview.asp#compiler)

RealView compiler is part of [Keil MDK](http://www2.keil.com/mdk5) development tools. The free version has a [limit](http://www2.keil.com/mdk5/selector/) of 32 KB of code. But the port has a larger size, so be sure you get a license. For example, [7-Day MDK-Professional Trial License](http://www2.keil.com/mdk5/activation/). You can read [here](http://www2.keil.com/mdk5/install/) about Keil MDK installation.

It should be installed by default in the folder `C:\Keil_v5`.

GCC is free, but it generates the firmware that has a 10% greater size than those generated by the RealView compiler. GCC ARM Embedded can be downloaded as a zip file and unzipped to any destination. I will call a folder with contents of zip file: `<gcc folder>`.


Compile Using the Arm Realview Compilation Tools
------------------------------------------------

The repository already has settings for MDK version 3.1, 3.80a, 4.12, 4.13, 4.54, 5.04, 5.05. If you want to use a different version, you can add a few lines in a file

   `<repo_dir>\tools\Targets\Microsoft.Spot.system.mdk.targets`

I used version 5.06. To do this after the lines

    <CC Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.05'">"$(MDK_TOOL_PATH)\ARMCC\bin\armcc.exe"</CC>
    <CPP     Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.05'">"$(MDK_TOOL_PATH)\ARMCC\bin\armcc.exe"</CPP>
    <AS      Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.05'">"$(MDK_TOOL_PATH)\ARMCC\bin\armasm.exe"</AS>
    <LINK    Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.05'">"$(MDK_TOOL_PATH)\ARMCC\bin\armlink.exe"</LINK>
    <AR      Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.05'">"$(MDK_TOOL_PATH)\ARMCC\bin\armar.exe"</AR>
    FROMELF Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.05'">"$(MDK_TOOL_PATH)\ARMCC\bin\fromelf.exe"</FROMELF>
    <MdkCrtLibLinkSwitch Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.05'">$(MdkCrtLibLinkSwitch) $(SWTC)libpath $(MDK_TOOL_PATH)\ARMCC\LIB</MdkCrtLibLinkSwitch>

I added lines

    <CC      Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.06'">"$(MDK_TOOL_PATH)\ARMCC\bin\armcc.exe"</CC>
    <CPP     Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.06'">"$(MDK_TOOL_PATH)\ARMCC\bin\armcc.exe"</CPP>
    <AS      Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.06'">"$(MDK_TOOL_PATH)\ARMCC\bin\armasm.exe"</AS>
    <LINK    Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.06'">"$(MDK_TOOL_PATH)\ARMCC\bin\armlink.exe"</LINK>
    <AR      Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.06'">"$(MDK_TOOL_PATH)\ARMCC\bin\armar.exe"</AR>
    <FROMELF Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.06'">"$(MDK_TOOL_PATH)\ARMCC\bin\fromelf.exe"</FROMELF>
    <MdkCrtLibLinkSwitch Condition="'$(COMPILER_TOOL_VERSION)'=='MDK5.06'">$(MdkCrtLibLinkSwitch) $(SWTC)libpath $(MDK_TOOL_PATH)\ARMCC\LIB</MdkCrtLibLinkSwitch>

Now you can start compiling. Open a command prompt and navigate to the folder with the repository, like this:

    cd /d D:\WORKDIR\NetMf\NetMFRepo\repo

then you need to set environment variables by running:

    setenv_mdk 5.06

![enter image description here](https://habrastorage.org/files/104/2d9/e34/1042d9e3466c445b98f05efe3e1eb9ff.png)

Then go to the folder with the port (`<repo_dir>\Solutions\STM32F4DISCOVERY`). For example:

    cd /d D:\WORKDIR\NetMf\NetMFRepo\repo\Solutions\STM32F4DISCOVERY

Now you can start the compilation by using, for example, the following command:

    msbuild dotnetmf.proj /p:flavor=release /fl

where

*msbuild* - starts compilation
*dotnetmf.proj*- STM32F4DISCOVERY port project
*/p:flavor=release* - build type (debug/release/rtm)
*/fl* - write log to the file.

The log file will be in the current folder (in my example path it is `D:\WORKDIR\NetMf\NetMFRepo\repo\Solutions\STM32F4DISCOVERY`). If the log file is not needed, */fl* can be removed.

To see all the options,  you need to execute this command:

    msbuild /t:help

![enter image description here](https://habrastorage.org/files/3cd/30b/7d7/3cd30b7d7f9d469fa52ae4cc81f5c74c.png)

Compiling takes about 10 minutes for me:

![enter link description here](https://habrastorage.org/files/d8c/f9f/e8b/d8cf9fe8b8ae4fa88e02bf14930992bc.png)

The result is a set of files but we will need only these:

    <repo_dir>\BuildOutput\THUMB2FP\MDK5.06\le\FLASH\release\STM32F4DISCOVERY\bin\Tinybooter.hex
    <repo_dir>\BuildOutput\THUMB2FP\MDK5.06\le\FLASH\release\STM32F4DISCOVERY\bin\tinyclr.hex\ER_CONFIG
    <repo_dir>\BuildOutput\THUMB2FP\MDK5.06\le\FLASH\release\STM32F4DISCOVERY\bin\tinyclr.hex\ER_FLASH​​​​​​


To “clean” build before command

    msbuild dotnetmf.proj /p:flavor=release /fl

execute command

    msbuild /t:clean

or remove folder

    <repo_dir>\BuildOutput

![enter image description here](https://habrastorage.org/files/2bc/d3d/835/2bcd3d83505942a69195dd635c031a58.png)


Compile using GCC ARM Embedded
------------------------------

Using GCC may require another fix. In the file:

    <repo_dir>\Solutions\STM32F4DISCOVERY\STM32F4DISCOVERY.settings

after the line

    <NO_BOOTLOADER_COMPRESSION>true</NO_BOOTLOADER_COMPRESSION>

you need to add

    <PLATFORM_EMULATED_FLOATINGPOINT Condition="'$(COMPILER_TOOL)'=='GCC'">true</PLATFORM_EMULATED_FLOATINGPOINT>

This fixes *“NNNN.a uses VFP register arguments”* issue. Read more details [here](https://github.com/NETMF/netmf-interpreter/pull/341).

However, this error can not occur if you use a "clean" build.

To “clean” build before command

    msbuild dotnetmf.proj /p:flavor=release /fl

execute command

    msbuild /t:clean

or remove folder

    <repo_dir>\BuildOutput

![enter image description here](https://habrastorage.org/files/2f9/6c2/6d2/2f96c26d28d54d6e8b5cca6674ab06c8.png)

So, to build the port, you need to open a command prompt and navigate to the folder with the repository. For example:

    cd /d D:\WORKDIR\NetMf\NetMFRepo\repo

Then you need to set environment variables by running:

    setenv_gcc <gcc ver> <gcc folder>

where 
*<gcc ver>* - GCC version
*<gcc folder>* - GCC ARM Embedded path

The command might look like this:

    setenv_gcc 4.9.3 D:\WORKDIR\NetMf\gcc_4_9_3

![enter image description here](https://habrastorage.org/files/05e/d52/15e/05ed5215e515412c91e468ccd5737790.png)

Then go to the folder with the port (`<repo_dir>\Solutions\STM32F4DISCOVERY`). For example:

    cd /d D:\WORKDIR\NetMf\NetMFRepo\repo\Solutions\STM32F4DISCOVERY

Now you can start the compilation by using, for example, the following command:

    msbuild dotnetmf.proj /p:flavor=release /fl

where

*msbuild* - starts compilation
*dotnetmf.proj* - STM32F4DISCOVERY port project
*/p:flavor=release* - build type (debug/release/rtm)
*/fl* - write log to the file.

The log file will be in the current folder (in my example path is `D:\WORKDIR\NetMf\NetMFRepo\repo\Solutions\STM32F4DISCOVERY`). If the log file is not needed, */fl* can be removed.

To see all the options, you need execute this command:

    msbuild /t:help

![enter image description here](https://habrastorage.org/files/01f/0d1/d28/01f0d1d2820948708cc4678ce6d87ecf.png)

Compiling takes about 10 minutes for me:

![enter image description here](https://habrastorage.org/files/c7a/165/c33/c7a165c33ccf4cf7b6127e48446dab75.png)

The result is a set of files but we will need only these:

    <repo_dir>\BuildOutput\THUMB2FP\GCC4.9\le\FLASH\release\STM32F4DISCOVERY\bin\Tinybooter.hex
    <repo_dir>\BuildOutput\THUMB2FP\GCC4.9\le\FLASH\release\STM32F4DISCOVERY\bin\tinyclr.hex\ER_CONFIG
    <repo_dir>\BuildOutput\THUMB2FP\GCC4.9\le\FLASH\release\STM32F4DISCOVERY\bin\tinyclr.hex\ER_FLASH


Uploading firmware to board
---------------------------

We have 3 files: `Tinybooter.hex`, `ER_CONFIG` и `ER_FLASH`. Tinybooter is a bootloader. It is used for CLR uploading. `ER_CONFIG` and `ER_FLASH` is CLR itself.

Also you will need additional software:

 1. [STM32 ST-LINK Utility](http://www.st.com/web/en/catalog/tools/PF258168#) - utility for TinyBooter upload
 2. Installed [MicroFraimworkSDK.MSI](https://github.com/NETMF/netmf-interpreter/releases/download/v4.4-RTW-20-Oct-2015/MicroFrameworkSDK.MSI) and [NetMF VS 14.vsix](https://github.com/NETMF/netmf-interpreter/releases/download/v4.4-RTW-20-Oct-2015/NetMFVS14.vsix) files. The first contains the required libraries and utilities, the second contains .net micro fraimwork template projects for Visual Studio.
 3. [USB driver](http://www.codeplex.com/Download?ProjectName=netmf4stm32&DownloadId=471395) needed for MicroFraimworkSDK utilities. Without these, the driver utilities will not recognize the board. (It is not required for Windows 10).

Do the following to upload firmware:

 1. Connect the board to the computer via the **mini USB** cable:

 ![enter image description here](https://habrastorage.org/files/87e/37e/027/87e37e0278304e0e8ef5258185d8640e.png)

 2. Run STM32 ST-LINK Utility and select the menu *Target-> Connect*

 ![enter image description here](https://habrastorage.org/files/3e5/65a/6fa/3e565a6fafb74a80a446c43d931715b2.png)

 After connecting with the board STM32 ST-LINK Utility will look like this

 ![enter link description here](https://habrastorage.org/files/59f/058/e39/59f058e39da94ea99b4241c2a87a36a8.png)

 3. It is necessary to delete the current firmware by selecting *Target-> Erase Sectors ...*

 ![enter image description here](https://habrastorage.org/files/340/078/865/3400788657474b8d93bd152e8fff4989.png)

 And there click *Select All* and then *Apply*

 ![enter link description here](https://habrastorage.org/files/041/04a/82f/04104a82f39547e9973dbbe192626abf.png)

 The microcontroller flash cleaning process:

 ![enter image description here](https://habrastorage.org/files/eff/a25/2f3/effa252f30dc4fa1a3da34fab83a684f.png)

 After cleaning STM32 ST-LINK Utility will look like this:

 ![enter image description here](https://habrastorage.org/files/019/50b/4ce/01950b4cecf941e69cc1acfe4e31d470.png)

 4. It is necessary to upload `TinyBooter.hex` by selecting *Target-> Program & Verify ...*

 ![enter image description here](https://habrastorage.org/files/667/05b/1b6/66705b1b6189409d81ce21a475162f1b.png)

 then select the `tinybooter.hex` file and press *Start*

 ![enter image description here](https://habrastorage.org/files/d08/a22/04d/d08a2204d59b4f26a49d2bdf3cdd5ce8.png)

 After firmware uploading STM32 ST-LINK Utility will look like this:

 ![enter image description here](https://habrastorage.org/files/908/6b2/dca/9086b2dca33747b5871301a0302b146c.png)

 5. It is necessary to restart the board by pulling out the **mini Usb** cable or pressing the black Reset button.

 6. Insert the **micro USB** cable:

 ![enter image description here](https://habrastorage.org/files/53d/34d/0cf/53d34d0cf1e44c7da6ba2e4a77e41681.png)

 Now STM32 ST-LINK Utility can be closed. **Mini Usb** cable will be used only as power supply.

 7. On Windows 10 the driver is installed automatically, but for other versions it is required to install the [USB driver](http://www.codeplex.com/Download?ProjectName=netmf4stm32&DownloadId=471395) manually.

 ![enter link description here](https://habrastorage.org/files/d93/cd5/318/d93cd53182744e36b3526e893a607a69.png)

 8. Now you need to run .NET Micro Framework Deployment Tool.

 You can find it in MicroFrameworkSDK:
 
 `C:\Program Files (x86)\Microsoft .NET Micro Framework\v4.4\Tools\MFDeploy.exe`

 It is necessary to switch from the Serial interface to USB:

 ![enter image description here](https://habrastorage.org/files/e6e/28e/4af/e6e28e4af2cb4186bbe4f5d63899217f.png)

 This will show the name of the board. You can click the Ping button to check TinyBooter. The console will display *Pinging ... TinyBooter*.

 ![enter image description here](https://habrastorage.org/files/e86/be2/fdc/e86be2fdc937462681798ac7b95f89a4.png)

 9. Next you need to upload remaining two files `ER_CONFIG` and `ER_FLASH` by using .NET Micro Framework Deployment Tool. Select them by clicking the lower *Browse...* button:

 ![enter image description here](https://habrastorage.org/files/682/7b6/d5d/6827b6d5d82643abb63b409515efb906.png)

 Click the *Deploy* button to upload the firmware.

 ![enter image description here](https://habrastorage.org/files/bd4/a20/c3b/bd4a20c3bc8e4405bce07afb4de31b95.png)

 You can press the *Ping* button again and make sure the CLR deployed on the board after uploading:

 ![enter image description here](https://habrastorage.org/files/119/062/996/1190629960a344a0a27d3b2a09c633a5.png)

 Now board is ready for work.

Create blinky project by Visual Studio
--------------------------------------

Now you can build and run the project in the Visual Studio. We make a simple project blinky flashing LEDs.

Start Visual Studio and create a new project:

![enter image description here](https://habrastorage.org/files/2c8/4fd/713/2c84fd7138d04975bdbb1fb0051b4c0f.png)

If SDK and vsix installation was performed correctly, there will be a new Micro Framework project template. Choose Console Application:

![enter image description here](https://habrastorage.org/files/34d/bcd/3f9/34dbcd3f9e734dfd8c6c0ca1b8023b18.png)

You can go to the project properties after solution creating:

![enter image description here](https://habrastorage.org/files/4e9/c11/fb9/4e9c11fb9c4742e2abc623b3e6bb4fde.png)

Select USB in the Transport field in the .NET Micro Framework project settings tab. After that, the name of the board should appear in the Device field:

![enter image description here](https://habrastorage.org/files/21e/ea3/0c8/21eea30c82b24c02983c82f2ab6287e2.png)

Save and close the configuration.

Next you need to add a Reference to the assembly at:

    C:\Program Files (x86)\Microsoft .NET Micro Framework\v4.4\Assemblies\le\Microsoft.SPOT.Hardware.dll

![enter image description here](https://habrastorage.org/files/a72/565/96a/a7256596a551489d8b1fdc1be52abc83.png)

And the last step is to modify the code in `program.cs`:

    using System;
    using System.Threading;
    using Microsoft.SPOT;
    using Microsoft.SPOT.Hardware;
    
    namespace STM32F4DISC_Test
    {
       public class Program
       {
           public static void Main()
           {
               OutputPort ledGreen = new OutputPort((Cpu.Pin)60, false);
               OutputPort ledYellow = new OutputPort((Cpu.Pin)61, false);
               OutputPort ledRed = new OutputPort((Cpu.Pin)62, false);
               OutputPort ledBlue = new OutputPort((Cpu.Pin)63, false);
    
               while (true)
               {
                   ledGreen.Write(true);
                   Thread.Sleep(500);
                   ledYellow.Write(true);
                   Thread.Sleep(500);
                   ledRed.Write(true);
                   Thread.Sleep(500);
                   ledBlue.Write(true);
                   Thread.Sleep(500);
                   ledGreen.Write(false);
                   ledYellow.Write(false);
                   ledRed.Write(false);
                   ledBlue.Write(false);
                   Thread.Sleep(500);
               }
           }
       }
    }

Run the project:

![enter image description here](https://habrastorage.org/files/eaf/a66/aa6/eafa66aa680249969b6b64012a2738a2.png)

And after a few seconds, the LEDs on the board will start blinking.

Summary
-------

.NET Micro Framework is a quite complex project . At the moment, it still requires certain skills and knowledge, especially when working with the repository. In this article, I specifically discussed as much as possible in detail what is encountered in building the ports. This information is useful for providing solutions to new boards.

However, you can easy use .NET Micro Fraimwork on STM32F4Discovery by taking the already existing files Tinybooter.hex, `ER_CONFIG` and `ER_FLASH`. You can download them [here](https://drive.google.com/file/d/0B4VR_8m5Yr74Yktja1ZuZ0NDVFk/view?usp=sharing).

Points of Interest
------------------

Here are some links about .Net Micro Framework:

 - GitHub [repository](https://github.com/NETMF)
 - Old .NetMF [repo](https://netmf.codeplex.com/) on Codeplex
 - .NetMF team [blog](http://blogs.msdn.com/b/netmfteam/)
 - .NetMF [website](http://www.netmf.com/)
 - .NetMF from [GHI Electronics](https://www.ghielectronics.com/technologies/netmf)
 - .NetMF [website](http://informatix.miloush.net/microframework/Home.aspx) from Jan Kučera
 - [Netduino](http://www.netduino.com/)
 - [Articles](https://geektimes.ru/search/?target_type=posts&q=%5B.net%20micro%20framework%5D&order_by=date) about .NetMF in Russion
