# Introduction #
Some weeks ago, a new version of the .Net Micro Framework [was published](http://blogs.msdn.com/b/netmfteam/archive/2015/10/21/net-micro-framework-4-4-is-now-available.aspx). It is a very interesting but very specific technology. And, like many open source-projects, it has some problems with documentation. So, it can take much time to run just a simple example.

In this article, I want to correct this deficiency and to talk about how to quickly and easily install everything you need to work with the emulator of .Net Micro Framework on Windows. Russian version of this article is [here](https://geektimes.ru/post/265986/).

![](https://habrastorage.org/files/f2e/cd9/aa9/f2ecd9aa9e21484c960a484a02059df1.png)



#Backgound#
This article focuses on the .Net Micro Framework version 4.4. Starting with this release, the project moved to GitHub. It consists of two branches:


- [llilum](https://github.com/NETMF/llilum) — the new platform, which compiles C# code directly to a microcontroller assembly code,
- [netmf-interpreter](https://github.com/NETMF/netmf-interpreter) — .Net Micro Framework interpreter, "classical" branch, which uses realisations of runtime environment for specific hardware platforms.

I will talk about netmf-interpreter, because this branch is best for practical use at the moment.

For a start it should be noted that netmf-interpreter 4.4 only works with Visual Studio 2015 Community, Pro and Ultimate editions.

# Start working with .NetMF 4.4 #
There are three different ways to start working with netmf-interpreter 4.4:

- Build installation files from the repository,
- Download installation files from GitHub releases page,
- Download installation files through Visual Studio 2015 Extensions and Updates.
 
## Build from the repository ##

This way is the most difficult one. But it allows to use the most recent code from the repository. Morecover the ability to work with repository will be useful when you need to make some changes in the emulator or to build your own "ports" on other hardware platforms.

You can find the repository [here](https://github.com/NETMF/netmf-interpreter/wiki/Getting%20Started). It can be downloaded as a zip-archive or can be obtained using git. Instructions on how to get the repository and build the installation files can be found 
[here](https://github.com/NETMF/netmf-interpreter/wiki/Getting%20Started). This article is based on these instructions. Versions of the repository associated with specific releases can be downloaded as a zip-archive from [here](https://github.com/NETMF/netmf-interpreter/releases).

Use these steps to get repository using git:


1. Create public copy of the repository in your GitHub account on the GitHub servers with fork command. All pull requests must come from a public GitHub repository.
2. Get a local copy of the repository by using clone command. For example:

    `git clone https://github.com/<your GitHub ID>/netmf-interpreter.git`

	**Important:** When you select a path to a local repository, it is necessary to make at least one parent folder. For example, `D:\NETMF\repo`, where the `repo` - folder for the repository.

3. Set up a local repository as the Upstream. Establishing the official repository as the "Upstream" repository for your local clone enables you to pull changes from the latest official commits into your repository and resolve merge issues locally before submitting a pull request. The command for establishing the Upstream remote is:


    `git remote add upstream https://github.com/NETMF/netmf-interpreter.git`


**Important:** When you unpack the repository from a zip-file, you also need to make at least one parent folder. For example, `D:\NETMF\repo`, where the `repo` - folder for the repository.

Unfortunately .NET Micro Framework v4.4 Release To Web (RTW) contains issues that do not allow you to immediately build the installation files from the repository. However, these issues can be easily fixed.`

When you get your own local copy of the repository, you need to do this steps:

1. Download [binary tools](http://netmf.github.io/downloads/build-tools.zip) zip file. This file contains the tools needed to build both the installation files and "ports" for devices. In the future, it is planned to remove these utilities, but they are still needed.
2. Unzip the contents of *binary tools* zip-file in the parent folder of the repository. For example, for the path 
`D:\NETMF\repo`, where the `repo` - the repository folder, the *bin* and *tools* folder must be in the folder `D:\NETMF`.
3. **Important:** In the file 
	
	`<repo folder>\Framework\Tools\BuildTasksInternal\BuildSigner\BuildSignerSpotBuild.csproj`

	in line 37 change

	`<HintPath>$(MSBuildProgramFiles32)\Microsoft Internal\Codesign.Submitter\CODESIGN.Submitter.dll</HintPath>`

	to

	`<HintPath>$(SPOROOT)\tools\x86\CODESIGN\CODESIGN.Submitter.dll</HintPath>`

	This is the first issue fix. Without such replacement, build of the repository will not succeed. This fix is needed only for .NET Micro Framework v4.4 Release To Web (RTW). In the future, the repository will already contain correct file. You can read [here](https://github.com/NETMF/netmf-interpreter/issues/333) about this issue.

4. You need to download the CMSIS library and put it in the folder `<repo folder>\CMSIS`. More information about version and links to the CMSIS library can be found in the file `<repo folder>\CMSIS\ReadMe.md`.
	
	[CMSIS](http://www.arm.com/products/processors/cortex-m/cortex-microcontroller-software-interface-standard.php) stands for Cortex Microcontroller Software Interface Standard. This library does not depend on a particular manufacturer and intend to work with the Cortex-M core. CMSIS is supplied and supported by kernel developers - the ARM company. Using this library allows you to simplify the creation of the "ports" for different microcontrollers from different manufacturers.

	In case of version of .Net Micro Framework 4.4 you need to download CMSIS version 4.3 or newer. The library is supplied as a zip-archive (CMSIS-SP-00300-r4p3-00rel0.zip). It can be downloaded at [ARM web site](http://www.arm.com/products/processors/cortex-m/cortex-microcontroller-software-interface-standard.php). The content of the archive should be put in the folder `<repo folder>\СMSIS`.

5. In this step you need to install the .Net Micro Framework Cryptographic Libraries. These libraries are used to sign the assemblies that will be executed on the microcontroller. Only executable files of cryptographic library are needed for build process. But you can download source code too.

	Libraries are available as msi file. I recommend to install them in any convenient folder (we will call it `<crypto install folder>`), and then copy them to the root of each repository. For example `D:\NETMF\repo` and `D:\NETMF\repo_master`.

	The separate folder with cryptographic libraries is needed to fix second build issue.

The repository and all the necessary libraries are finally downloaded. You can build the installation files. .Net Micro Framework is using [MSBuild](https://ru.wikipedia.org/wiki/MSBuild) as a building platform. The repository contains many sln and proj files, but Visual Studio can’t be used because these files contain extended format. Visual studio uses MSBuild too, but this building platform allows to create more complex projects.

So, you need to open a command prompt and navigate to the folder with the repository, like this:

`cd /d D:\WORKDIR\NetMf\NetMFRepo\repo`

Then you need to execute `build_sdk.cmd` file. This will start build process:

![](https://habrastorage.org/files/e97/73b/62e/e9773b62eed043afa934c98cc3df8f2d.png)

Build process will be completed after a few minutes. Many files will be created, but we need only one: `<repo folder>\BuildOutput\public\Release\Server\dll\NetmfVS14.vsix`

However, the second installation file will not be created due to the second issue. The final step is fixing it. You need to copy file

	<crypto install folder>\crypto\lib\x86\dll\crypto.dll

into folder

	repo folder>\BuildOutput\public\Release\Server\dll\

This library should be copied automatically while building process, but this is not done.

If you start build process again, there will be created two files:

	<repo folder>\BuildOutput\public\Release\Server\dll\NetmfVS14.vsix
	<repo folder>\BuildOutput\public\Release\Server\msm\MicroFrameworkSDK.MSI


Now you have all installation files.

`NetmfVS14.vsix` is Visual Studio plugin. `MicroFrameworkSDK.MSI` is libraries and emulator installation file.

Plugin will configure itself automatically when you install it. Libraries must be installed in the Typical setup type:


![](https://habrastorage.org/files/266/55c/545/26655c5459b9449e844f0409457b5a5c.png)


## Installation from Visual Studio ##

`MicroFrameworkSDK.MSI` and `NetmfVS14.vsix` files can be obtained by using *Tools->Extensions and Updates* menu item in Visual Studio:

![](https://habrastorage.org/files/37a/e91/bdb/37ae91bdb47d43f0b01ca21bdba16b72.png)

Select *Online* tab in opened dialog and use “netmf” keyword in the search field (red frame):

![](https://habrastorage.org/files/b2c/b32/046/b2cb320464114e34a7323cca5f3b9bc5.png)

Green frame highlight what you need to download and install.

`.NET Micro Framework project system` is Visual Studio plugin. It matches `NetmfVS14.vsix` file and is install through *Extensions and Updates* dialog.

`.NET Micro Framework SDK` is libraries and emulator installation file. This is a `MicroFrameworkSDK.MSI` file. You need to download and install it in *Typical* setup type:

![](https://habrastorage.org/files/266/55c/545/26655c5459b9449e844f0409457b5a5c.png)

## Downloading from GitHub ##

*MicroFrameworkSDK.MSI* and *NetmfVS14.vsix* files can be downloaded from releases page:

![](https://habrastorage.org/files/0de/d4c/98d/0ded4c98d89f472f93658968f71a2f3c.png)

*NetmfVS14.vsix* is Visual Studio plugin. *MicroFrameworkSDK.MSI* is libraries and emulator installation file.

Plugin will configure itself automatically when you install it. Libraries must be installed in the *Typical* setup type:

![](https://habrastorage.org/files/266/55c/545/26655c5459b9449e844f0409457b5a5c.png)

# Create and launch first project #

Now all libraries and plugins are installed by one of the three ways. So, you can create your first project on the .Net Micro Framework.

The Visual Studio plugin already has some examples. You need only to choose which one you like.

However, there is one important thing. The emulator is installed on the system disk and need to be run in an administrator mode. Otherwise, it will not work correctly. The emulator is started from the Visual Studio, so exactly Visual Studio must be run in administrator mode.

Run Visual Studio as an administrator and create a new project:

![](https://habrastorage.org/files/749/451/936/7494519368ee4a6e96ed392261c4d9b0.png)

If the installation was performed correctly, there will be a new Micro Framework project template:

![](https://habrastorage.org/files/c6d/fbc/4c2/c6dfbc4c2f094acf85e6c8d77101654c.png)

Choose Windows Application as the most interesting project. You can go to the project properties after creating a solution:

![](https://habrastorage.org/files/064/2df/275/0642df275cf746d2a8d3279b8d2a5fa9.png)

You can find a new tab .NET Micro Framework in the project settings:

![](https://habrastorage.org/files/757/0fe/18f/7570fe18f14e427f9f3a6faf14ce905a.png)

Leave all the default values to work with the emulator. If you build and run the project, then you will see the emulator window:

![](https://habrastorage.org/files/117/4ff/49e/1174ff49e50047198d1382deab4d8c8f.png)

The emulator have a screen, 5 buttons, two virtual SD-cards and a COM-port. All of these perfectly work. For example, pressing the buttons will display their code in the Visual Studio Output window:

![](https://habrastorage.org/files/af5/20d/39b/af520d39b3534e788a9067df238c0ce1.png)

You can modify the project and try other functionality.

# Summary #

Net Micro Framework is a promising open-source technology that allows you to write C# code for microcontrollers. With new [IoT](https://ru.wikipedia.org/wiki/%D0%98%D0%BD%D1%82%D0%B5%D1%80%D0%BD%D0%B5%D1%82_%D0%B2%D0%B5%D1%89%D0%B5%D0%B9) ideas and creation [UWP](https://msdn.microsoft.com/ru-ru/library/windows/apps/dn958439.aspx) concept its development received a new impulse. With time it will cover very small devices that can’t support Windows 10 IoT, but together with this fully support the UWP concept.

# Points of Interest #

Here are some links about .Net Micro Framework:

- GitHub [repository](https://github.com/NETMF).
- Old .NetMF [repo](https://netmf.codeplex.com/) on Codeplex
- .NetMF team [blog](http://blogs.msdn.com/b/netmfteam/)
- .NetMF [website](http://www.netmf.com/)
- .NetMF from [GHI Electronics](https://www.ghielectronics.com/technologies/netmf)
- .NetMF [website](http://informatix.miloush.net/microframework/Home.aspx) from *Jan Kučera*
- [Netduino](http://www.netduino.com/)
- [Articles](https://geektimes.ru/search/?target_type=posts&q=%5B.net%20micro%20framework%5D&order_by=date) about .NetMF in Russion
