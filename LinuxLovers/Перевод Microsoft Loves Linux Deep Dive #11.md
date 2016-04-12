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

 ![enter image description here](https://habrastorage.org/files/9ad/8e9/962/9ad8e9962a37412c89f9805a1cb50dc9.jpg)

DSC для Linux Local Configuration Manager является провайдером Common Information Model (CIM), зарегистрированным с помощью Open Management Infrastructure (OMI) сервером. OMI это не требовательный к ресурсами open-source CIM сервер, который был рассмотрен в предыдущих статьях этой серии: System Center [Operations Manager](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/11/10/microsoft-loves-linux-deep-dive-8-linux-and-unix-monitoring-with-operations-manager.aspx) и агенты [Configuration Manager](https://blogs.technet.microsoft.com/b/server-cloud/archive/2015/12/03/microsoft-loves-linux-deep-dive-10-managing-the-software-configuration-on-linux-and-unix-servers.aspx) для UNIX и Linux. Ресурсы также зарегистрированы с помощью OMI, но вызываются они LCM провайдером а не на прямую OMI сервером. На текущий момент OMI и DSC для Linux поддерживают только нативные (то есть написанные на C или C++) ресурсы. Однако, доступные сейчас ресурсы из комплекта поставки и скачиваемые ресурсы написаны на Python. Это позволяет создать тонкую C++ прослойку для каждого ресурса, которая использует сокеты для взаимодействия с клиентским Python слоем.

Конфигурации могут быть отправлены на DSC для Linux через протокол Web Services Management (WSMan) (через сессию PowerShell CIM или любой другой WSMan). В этой модели соединение WSMan выполнено с  OMI сервером. Кроме того, конфигурации могут быть загружены с DSC pull-сервера, как вашего собственного, так и сервиса [Azure Automation DSC](https://azure.microsoft.com/en-us/documentation/articles/automation-dsc-overview/).  При использовании pull модели LCM периодически проверяет изменения конфигурации на pull-сервере и правильность применения текущей конфигурации. В pull операциях применяется обычный cURL клиент для подключения к web сервису.

##Ресурсы##

Пакет DSC для Linux включает в себя 10 ресурсов:

- nxArchive – синхронизирует .tar/.zip архивы с директорией
- nxFile – управляет файлами и директориями
- nxFileLine – управляет строками в файле
- nxPackage – управляет установленными пакетами
- nxUser  – управляет локальными аккаунтами пользователей
- nxGroup – управляет группами пользователей
- nxScript – расширенный провайдер для user-defined Get/Set/Test скриптов
- nxEnvironment – настраеивает переменные окружения
- nxSshAuthorizedKeys – управляет публичными ssh ключами для пользователей

Два дополнительных ресурсных модуля уже доступны в PowerShell Gallery: nxComputerManagement и nxNetworking. Они добавляют четыре дополнительных ресурса:

- nxComputer – управляет именем хоста, временной зоной и доменным именем DNS
- nxIPAddress – управляет конфигурацией IPv4/IPv6
- nxDNSServerAddress –управляет адресами DNS сервером (DNS клиент)
- nxFirewall – управляет правилами firewall для 6 распространенных типов firewall для Linux

##Azure Automation DSC##

[Azure Automation DSC](https://azure.microsoft.com/en-us/documentation/articles/automation-dsc-overview/) это построенный по модели pull-сервера облачный сервис, который заметно упрощает централизованное управление конфигурациями Windows и Linux (с помощью DSC конфигураций) и позволяет получать расширенные отчеты о соответствии конфигурации на управляемых серверах. [Регистрация Linux сервера](https://azure.microsoft.com/en-us/documentation/articles/automation-dsc-onboarding/#physical-virtual-linux-machines-on-premises-in-azure-or-in-a-cloud-other-than-azure) для работы с Azure Automation DSC требует всего лишь установки пакета DSC для Linux (и OMI) и выполнения одной команды. С помощью портала Azure или с помощью Azure PowerShell вы можете загружать DSC конфигурации, компилировать их в MOF документы и назначать их зарегистрированным серверам. Вы можете посмотреть историю статистики соответствия для управляемых систем сразу после назначения конфигурации:

![enter image description here](https://habrastorage.org/files/c62/26c/34d/c6226c34d9734b2caf5891b483f288f2.jpg)
 

##Azure DSC для Linux##

*Расширение* Azure VM доступно и для DSC для Linux. [Это расширение](https://azure.microsoft.com/en-us/blog/introduce-azure-dscforlinux-extension/) установит DSC для Linux и применит указанную вами конфигурацию (MOF) вместе с заданными ресурсными модулями. Оно может быть развернуто через Azure CLI или Azure PowerShell и может быть встроено в ваши ARM шаблоны. Это отличный способ автоматизации начальной конфигурации ваших виртуальных машин Linux или развертывания приложений в Azure IaaS.  Полную документацию по расширению DSC для Linux можно найти в [репозитории на GitHub](https://github.com/Azure/azure-linux-extensions/tree/master/DSC). Подведем итог: Azure Automation DSC представляет собой решение для непрерывного управления конфигурациями ваших Linux серверов (в не зависимости от того, где они запущены) с функциями исправления возникающих различий конфигураций и создания расширенных отчетов. Расширение Azure VM позволяет вам легко разворачивать пакеты DSC для Linux делать разовые настройки конфигурации для виртуальных машин Azure IaaS.

##Заключение##

PowerShell Desired State Configuration представляет собой гибкую декларативную платформу, которая может быть использована во множестве сценариев для автоматизации и управления серверными конфигурациями. PowerShell DSC для Linux переносит этот функционал на Linux и дает возможность одинакового управления конфигурациями для ваших разнородных дата-центров и облаков. Начать работать с DSC для Linuxможно с прочтения [Release Notes](https://www.microsoft.com/en-us/download/details.aspx?id=49150) и [документации](https://msdn.microsoft.com/en-us/powershell/dsc/lnxgettingstarted).

На следующей неделе мы подведем итог того, что предлагает Microsoft для запуска и управления Linux и UNIX на вашем локальном дата-центре и рассмотрим некоторые планы на будущее.
