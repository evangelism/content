*Статью написал Kristopher Bash, Senior Program Manager, Microsoft. Перевел Alexandr Surkov.*

#Введение#

Это 11 статья из цикла публикаций, посвященных запуску, настройке и управлению операционными системами Linux и FreeBSD на локальном датацентре. Другие статьи можно прочитать по ссылкам ниже:

##Обзор##

- [Microsoft Loves Linux](https://blogs.technet.microsoft.com/b/windowsserver/archive/2015/05/06/microsoft-loves-linux.aspx)

##Запуск Linux и FreeBSD как гостевых операционных система на Hyper-V##

 - [Microsoft Loves Linux Deep Dive #1: Introduction to Linux and FreeBSD on Hyper-V](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/09/22/microsoft-loves-linux-deep-dive-1-introduction-to-linux-and-freebsd-on-hyper-v.aspx) 
 - [Microsoft Loves Linux Deep Dive #2: Linux and FreeBSD Integration Services Core Features ](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/09/30/microsoft-loves-linux-deep-dive-2-linux-and-freebsd-integration-services-core-features.aspx)   
 - [Microsoft Loves Linux Deep Dive #3: Linux Dynamic Memory and Live Backup](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/10/07/microsoft-loves-linux-deep-dive-3-linux-dynamic-memory-and-live-backup.aspx) 
 - [Microsoft Loves Linux Deep Dive #4: Linux Network Features and Performance](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/10/13/microsoft-loves-linux-deep-dive-4-linux-network-features-and-performance.aspx) 
 - [Microsoft Loves Linux Deep Dive #5: FreeBSD Overview and Status](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/10/20/microsoft-loves-linux-deep-dive-5-freebsd-on-hyper-v.aspx)

##Управление Linux и UNIX с помощью System Center и PowerShell DSC##

 - [Microsoft Loves Linux Deep Dive #6:  Linux and UNIX Systems Management Overview](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/10/27/microsoft-loves-linux-deep-dive-6-linux-and-unix-systems-management-overview.aspx)
 - [Microsoft Loves Linux Deep Dive #7:  Managing Linux in System Center Virtual Machine Manager](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/11/03/microsoft-loves-linux-deep-dive-7-managing-linux-in-system-center-virtual-machine-manager.aspx)
 - [Microsoft Loves Linux Deep Dive #8:  Linux and UNIX Monitoring with Operations Manager](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/11/10/microsoft-loves-linux-deep-dive-8-linux-and-unix-monitoring-with-operations-manager.aspx)
 - [Microsoft Loves Linux Deep Dive #9:  Monitoring Open Source Middleware](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/12/03/microsoft-loves-linux-deep-dive-10-managing-the-software-configuration-on-linux-and-unix-servers.aspx)
 - [Microsoft Loves Linux Deep Dive #10:  Managing the Software Configuration on Linux and UNIX Servers](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/12/03/microsoft-loves-linux-deep-dive-10-managing-the-software-configuration-on-linux-and-unix-servers.aspx)
 - [Microsoft Loves Linux Deep Dive #11:  Linux Configuration Management with PowerShell DSC](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/12/10/microsoft-loves-linux-deep-dive-11-linux-configuration-management-with-powershell-dsc.aspx)
 - [Microsoft Loves Linux Deep Dive #12:  Summary of Running and Managing Linux and FreeBSD in Your Datacenter](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/12/17/microsoft-loves-linux-deep-dive-12-summary-of-running-and-managing-linux-and-freebsd-in-your-datacenter.aspx)

#Управление конфигурацией Linux с помощью PowerShell DSC#

PowerShell Desired State Configuration (DSC) это декларативная платформа для работы с конфигурациями впервые появившаяся в Windows Server 2012 R2. Наверное вы и не подозревали, что DSC может быть использована и для работы с серверами на Linux? PowerShell DSC для Linux впервые была выпущена в мае 2015 года. Текущая версия (1.1) доступна как в виде [пакетов для Linux](https://www.microsoft.com/en-us/download/details.aspx?id=49150) (rpm/deb), так и в виде [исходного кода](https://github.com/Microsoft/PowerShell-DSC-for-Linux). С помощью простого, удобного для восприятия человеком декларативного синтаксиса можно контролировать файлы и директории, управлять установленным программным обеспечением, настраивать службы и множество других аспектов серверной конфигурации. Текущее состояние конфигурации можно сравнить с любым выбранным состоянием и разница между ними может быть автоматически исправлена.

#Сценарии#

DSC является мощной и гибкой платформой для управления конфигурациями компьютеров с операционными системами Windows и Linux. Ключевые варианты использования DSC для Linux включают в себя:

 - **Развертывание виртуальных машин (VM)**: развертывание агента DSC для Linux на вашей виртуальной машине или шаблоне Azure Resource Manager (ARM) и автоматизация начальных параметров конфигурации и установки приложений с помощью заранее определенных MOF или по средствам pull-сервера или через Azure Automation DSC (подробнее об этом позже в этой статье).
 - **Настройка соответствия**: определение конфигурации в соответствии с применяемыми политиками настроек, обеспечивающих соответствие конфигурации, и тестирование/составление отчетов по состоянию.
 - **DevOps**: использование кода для определения конфигурации сервера и развертывание этой конфигурации с соответствующими приложениями.
 - **Непрерывное развертывание**: определение конфигурации для приложения и автоматическое развертывание новых релизов для всех серверов в ферме. Это сделано за счет возможности nxFile хранить файл или директорию синхронизированными с файлом-источником или директорией-источником. Точно так же nXArchive умеет синхронизировать файл архива с разархивированной директорией.

Работа с DSC для Linux начинается с **создания** конфигурации. Хотя и возможно использование любых инструментов или текстовых редакторов для создания конфигурационного документа (документа MOF), использование PowerShell значительно упрощает этот процесс за счет Domain Specific Language. Работа в PowerShell позволяет использовать декларативный синтаксис DSL (то есть свойство = значение) совместно с IntelliSence и валидацией схемы. При этом вы можете принимать свойства конфигурации как параметры и использовать богатый язык PowerShell для автоматизации создания MOF.

Простой пример конфигурации, проверяющей что установлены все модули для MariaBD и запущен соответствующий демон на CentOS или RHEL 7 может выглядеть так:

![enter image description here](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/prod.evol.blogs.technet.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/89/15/Linux121015.JPG)

Следующими шагами после создания MOF будут его размещение и развертывание. Развертывание может быть сделано двумя путями: публикация конфигурации на Linux сервере (в PowerShell это делается с помощью команды [Start-DSCConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx)) или размещения на pull сервере с последующей загрузкой DSC агентом в целевой Linux сервер.

В финальной фазе DSC агент на целевом Linux сервере разберет конфигурацию, протестирует текущую конфигурацию на совместимость с выбранной и выполнит изменения в системе в соответствии с выбранной конфигурацией. После того, как конфигурация была применена можно протестировать состояние системы на соответствие локально (выполнив TestDscConfiguration.py) или удаленно с помощью PowerShell (Test-DscConfiguration) Likewise. Подробные данные о текущей конфигурации могут быть запрошены локально (GetDscConfiguration.py) или удаленно с помощью PowerShell (Get-DSCConfiguration). Если вы используете Pull сервер или Azure Automation DSC, то отчет о состоянии соответствия конфигурации будет отправлен на сервер. Вы можете контролировать автоматическое исправление разницы между конфигурациями изменяя настройку [Meta Configuration](http://blogs.msdn.com/b/powershell/archive/2013/12/09/understanding-meta-configuration-in-windows-powershell-desired-state-configuration.aspx) ConfigurationMode у DSC агента. Если ConfigurationMode установлено в ApplyAndAutoCorrect, DSC агент будет периодически тестировать конфигурацию на соответствие, и, в случае обнаружения проблем, изменять различающиеся настройки конфигурации. Агент также может работать в режиме тестирования. Он будет отправлять отчеты о несоответствии конфигурации но не будет пытаться их исправить.

#Архитектура#

Программа DSC для Linux реализует модель поставщика и включает в себя два основных компонента: The Local Configuration Manager (LCM) и ресурсы. The LCM - это ключевой DSC агент, который должен быть установлен на каждом Linux сервере, предназначенном для управления с помощью DSC. Он обрабатывает конфигурационные документы и применяет сущности и операции к ресурсам. Ресурсы - это провайдеры, осуществляющие управление специфическими зонами, такими как файлы, демоны, пакеты и т.д. Ресурсы исполняют действия, указанные в конфигурации для того чтобы протестировать, получить и установить эту конфигурацию. Кроме того они образуют слой абстракции. Это очень ценно для Linux, различные дистрибутивы которого могут иметь разные пути установки одной и той же конфигурации. Абстракция позволяет скрывать тонкости реализации разных дистрибутивов. Теперь вы можете не ломать голову над синтаксисом конфигурационных файлов таких компонентов как сетевые адаптеры, фаерволы и демоны, который может различаться в разных версиях или дистрибутивах системы. Пакет DSC для Linux уже включает в себя набор встроенных ресурсов и позволяет установить дополнительные ресурсные модули, которые поставляются в виде .zip фалов и содержат схемы ресурсов и исполняемые файлы. При необходимости эти ресурсные модули могут быть скачаны с pull сервера или установлены локально с помощью InstallModule.py. 

##Схема архитектуры DSC для Linux##

 

The DSC for Linux Local Configuration Manager is a Common Information Model (CIM) provider, registered with an Open Management Infrastructure (OMI) server. OMI is an open source, lightweight CIM server and has been discussed in previous blog posts in this series pertaining to System Center Operations Manager and Configuration Manager agents for UNIX and Linux. The resources are also registered as providers with OMI, but they are invoked by the LCM provider, and not the OMI server directly.  Only native (i.e. written in C or C++) resources are currently supported by OMI and DSC for Linux. However, the built-in and downloadable resources presently available for DSC for Linux are authored in Python. This is facilitated through the use of a thin C++ layer for each resource that uses sockets to communicate with a client Python layer.

Configurations can be pushed to DSC for Linux over the Web Services Management (WSMan) protocol (with a PowerShell CIM session or any other WSMan client). In this model, the WSMan connection is made to the OMI server. Alternatively, configurations can be pulled from a DSC pull server, either your own pull server or the Azure Automation DSC service. In the pull model, a cron job periodically triggers the LCM to check for changed configurations on the Pull server and check that the current configuration is correctly applied. For pull operations, the ubiquitous cURL https client is used to connect to the web service.

Resources

The DSC for Linux software includes 10 built-In resources:

•  nxArchive – synchronize a .tar/.zip with a directory
•  nxFile – manage files & directories
•  nxFileLine – manage lines within a file
•  nxPackage – manage installed packages
•  nxUser  – manage (local) user accounts
•  nxGroup – manage user groups
•  nxScript – extensible provider with user-defined Get/Set/Test scripts
•  nxEnvironment – control defined environment variables
•  nxSshAuthorizedKeys – manage ssh (public) keys for users

Two additional resource modules are currently available on the PowerShell Gallery: nxComputerManagement and nxNetworking. These modules provide four additional resources:

•  nxComputer – manage hostname, timezone, DNS domain name
•  nxIPAddress – manage IPv4/IPv6 configuration for an interface
•  nxDNSServerAddress –manage DNS server addresses (DNS client)
•  nxFirewall – manage firewall rules for 6 common Linux firewalls

Azure Automation DSC

Azure Automation DSC is a cloud service built on the DSC pull server model that greatly simplifies centralized management of Windows and Linux configuration (with DSC configurations) and provides rich reporting on configuration compliance for managed servers. Registering a Linux server with Azure Automation DSC requires simply installing the DSC for Linux package (and OMI) and running a single command. Within the Azure portal, or with Azure PowerShell, you can upload DSC configurations, compile them to MOF documents, and assign the MOFs to registered servers. Once a configuration has been assigned, you can see the current and historical compliance status for the managed systems:

 

Azure DSC for Linux

An Azure VM extension is also available for DSC for Linux. This VM extension will install DSC for Linux and apply the configuration (MOF) that you specify as well as any resource modules you supply. The extension is deployable through Azure CLI or Azure PowerShell, can be embedded in your ARM templates, and is a great way to automate initial configuration of your Linux VMs or provisioning of applications in Azure IaaS. For full documentation of the DSC for Linux extension, visit the extension’s GitHub repo. To recap: Azure Automation DSC provides a solution for ongoing configuration management of your Linux servers (no matter where they run) with drift remediation and reporting, while the Azure VM Extension enables easy deployment of the DSC for Linux software itself and one-time configuration setting for Azure IaaS virtual machines.

Summary

PowerShell Desired State Configuration is a flexible declarative configuration tool that can be used in many scenarios to automate and manage server configurations. PowerShell DSC for Linux extends this capability to Linux and enables common configuration management for your heterogeneous data centers and clouds. To get started with DSC for Linux, check out the Release Notes, and documentation.

Next week, this blog series will wrap up with a summary of Microsoft’s offerings for running and managing Linux and UNIX in your on-premises datacenter, putting all the pieces in context and looking at some future directions.
