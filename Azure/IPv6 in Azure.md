<blockquote><p><em>Выражаем благодарность за подготовку статьи Михаилу Тряхову (<a href="https://habrahabr.ru/users/PerseptronYar/">@PerseptronYar</a>) из компании <a href="https://vk.com/akvelon_inc" >Akvelon (Ярославль)</a> за помощь в написании данной статьи. Михаил работает в команде разработчиков Microsoft Azure CLI (Command Line Interface) со специализацией на Networking Services. </em></p></blockquote>

Всем привет! 
Сегодня настал момент поговорить о долгожданном нововведении, которое, с дымом и спешкою, подготовили-таки к конференции Ignite. Полным ходом это событие идет с 28 по 30 сентября в Атланте, и о нем уже распрекрасно написано, скажем, [здесь](https://habrahabr.ru/company/microsoft/blog/311002/). Тем не менее уже известно, что IPv6 стал доступен в целом ряде регионов (location) Microsoft Azure. 

Сказать, что данный функционал ожидаем - ничего не сказать. Стоит, для начала, вбить в поисковике "Azure IPv6". Анонсирую: найдете вы лишь пару-тройку ссылок, в основном крутящихся вокруг Feature requests (например, [здесь](http://serverfault.com/questions/393297/does-windows-azure-present-an-ipv6-address) и [тут](https://feedback.azure.com/forums/217313-networking/suggestions/4992369-support-ipv6-throughout-the-azure-platform)) и [обещаний компании](http://stackoverflow.com/questions/31581816/ipv6-support-for-azure). 
И вот, свершилось!

![](https://acom.azurecomcdn.net/80C57D/cdn/mediahandler/docarticles/dpsmedia-prod/azure.microsoft.com/en-us/documentation/articles/load-balancer-ipv6-overview/20160926093315/load-balancer-ipv6.png)

<habracut/>

IPv4 и IPv6 - совершенно разные и никаким образом не зависящие друг от друга версии IP. То есть подразумевается, что невозможно конвертировать IPv4 адрес в IPv6 и т.п. Основной задачей, которую IPv6 начинает решать - обеспечение взаимодействия виртаульных машин Azure и IPv6-клиентов, интегрируя это с многообразием доступных в NRP (Network Resource Provider) сервисов, таких, как Load balancer (балансировщик нагрузки), NIC (Network Interface, интерфейс сети), public ip (публичные IP-адреса) и производные.

Не могу не предложить, ради интереса, попробовать в деле часть описываемых в посте команд на платформе Azure CLI. Как я уже [писал](https://habrahabr.ru/company/microsoft/blog/278175/) в первой обзорной статье в данном блоге - это расширение командной строки, позволяющее управлять инфраструктурой Azure на множестве операционных систем (Windows, Mac, многие дистрибутивы Linux). Чтобы установить ее, можно воспользоваться [инструкцией](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/) на официальном сайте. Обратите внимание, что установленная версия должна быть выпущена после 28 сентября 2016 года. Нужный релиз доступен [по ссылке](https://github.com/Azure/azure-xplat-cli/releases/tag/v0.10.5-September2016).

Порисуем. На картинке изобразим синим имевшиеся доселе возможности кастомизации балансировщика, зеленым - обновления, которые стали доступны. 

![](http://image.prntscr.com/image/0e873c1573e44c1ebac84323a220372b.png)

Опишу основные плюшки, которые нам это несет:
* Радость у IOT-разработчиков и не только - поддержка "dual stack" (IPv4+IPv6)
* Azure DNS отныне поддерживает IPv4 наборы записей типа A и IPv6 типа AAAA  
* Многие обновленные образы виртуальных машин с операционной системой Linux уже поддерживают данный функционал из коро. В некоторых случаях, необходимо обратиться к [<b>DHCP</b>](https://azure.microsoft.com/ru-ru/documentation/articles/load-balancer-ipv6-for-linux/) 

Существует и ряд ограничений. Например, произвести данные модификации по использованию IPv6 мы уже не сможем через графический интерфейс. Как я уже писал раньше, работать с JSON и консолями придется больше и больше. Так и здесь, вариантов остается немного:
* Cоздать JSON-шаблон (template) и [**импортировать**](https://azure.microsoft.com/ru-ru/documentation/articles/load-balancer-ipv6-internet-template/) ее в соответствующие группы ресурсов
* [**Использовать PowerShell**](https://azure.microsoft.com/ru-ru/documentation/articles/load-balancer-ipv6-internet-ps/) 
* [**Использовать Azure XPlat CLI**](https://azure.microsoft.com/ru-ru/documentation/articles/load-balancer-ipv6-internet-ps/)

Для начала, в выбранной группе ресурсов (речь, как Вы поняли, идет исключительно об ARM), необходимо настроить сеть, подсети и создать "пустой" load balancer. Средствами Azure CLI для этого достаточно выполнить команды из упомянутого выше [**руководства**](https://azure.microsoft.com/ru-ru/documentation/articles/load-balancer-ipv6-internet-cli/). Я буду опираться на приведенный там (как и с PowerShell, template) пример, сконцентрировавшись на важных (с точки зрения данной темы) моментах. Цель последующих абзацев - максимально подробно разобрать процесс развертывания указанной архитектуры и описать имеющиеся ограничения.

Поскольку все три доступных нам способа обеспечить поддержку IPv6 не мытьём, так катаньем сводятся к тому, что лишь отправляют соответствующей API-версии Azure SDK сформированный JSON, то именно работу с JSON-шаблонами я и считаю наиболее верным демонстрировать, сосредоточившись на нововведениях.


### Настройка Public IP 

Итак, согласно нарисованной выше схеме, мы делаем PUT запросы и получаем два public IP:

```json
{
    "name": "myIPv4Vip",
    "id": "/subscriptions/{guid}/resourceGroups/rg1/Microsoft.Network/publicIpAddresses/ip1",
    "location": "West US",
    "tags": { “key”: “value” } ,
     “etag”: “W/\”00000000-0000-0000-0000-000000000000\”“,
 
    "properties": { 
        "provisioningState": “Updating|Deleting|Failed|Succeeded”,
        "ipAddress": "1.1.1.1",
        "publicIpAddressVersion": IPv4,
        "publicIPAllocationMethod": "Static | Dynamic",
        "idleTimeoutInMinutes": 4,
        "ipConfiguration": { "id": "/subscriptions/{guid}/../Microsoft.Network/loadbalancers/MyIpV4AndV6LB1/ipConfigurations/frontendIP1"},
        "dnsSettings": 
        { 
            "domainNameLabel": "mylabel",
            "fqdn": "mylabel.westus.cloudapp.azure.com.",
            "reverseFqdn": "contoso.com."
        }
    }
}
```

Здесь прошу обратить внимание на ряд моментов
* Приведенный **location** - один из немногих, поддерживаемых на данный момент. Я правда очень стараюсь отрыть источник с актуальными данными по этому вопросу.
* **publicIpAddressVersion = IPv4** идет по умолчанию

```json
{
    "name": "myIPv6Vip",
    "id": "/subscriptions/{guid}/resourceGroups/rg1/Microsoft.Network/publicIpAddresses/myIPv6Vip",
    "location": "West US",
    "tags": { “key”: “value” } ,
     “etag”: “W/\”00000000-0000-0000-0000-000000000000\”“,
 
    "properties": { 
        "provisioningState": “Updating|Deleting|Failed|Succeeded”,
        "ipAddress”:”2015::1234:5":   ,
        "publicIpAddressVersion": IPv6,
        " publicIPAllocationMethod ": " Static|Dynamic",
        "idleTimeoutInMinutes  ": 4,
        "ipConfiguration": { "id": "/subscriptions/{guid}/../Microsoft.Network/loadbalancers/MyIpV4AndV6LB1/ipConfigurations/frontendIP2"},
        "dnsSettings": 
        { 
            "domainNameLabel": "mylabel",
            "fqdn": "mylabel.westus.cloudapp.azure.com.",
            "reverseFqdn": 
        },
    }
```

Здесь есть ряд ограничений
* **dnsSettings.fqdn** - доступное только для чтение свойство в формате FQDN, то есть в нашем примере - **contoso09152016.westus.cloudapp.azure.com**.
* **DNS settings  reverse FQDN** не указывается, поскольку на данный момент (и в ближайшей перспективе) Azure DNS не планирует поддерживать обратные запросы для IPv6. Значение будет пустым (null/not allowed).
* **IP адрес** будет доступен только для чтения
* **Public IP Allocation Method**: При указании **Static**  будет создан зарезервированный виртуальный IP-адрес (reserved VIP), то есть VIP остается неизменным в балансировщиках нагрузки. Если же параметр указан как **Dynamic**, то значение IP-адреса может измениться при привязывании данного адреса к другому балансировщику нагрузки.
* Использовать IPv6 можно только при создании новых экземпляров Public IP. Изменять уже существующие с IPv4 на IPv6, увы, невозможно.

### Настройка Load Balancer

Следующим важным шагом в разборе приведенной архитектуры будет являться настройка балансировщика нагрузки. Он будет ожидаемо большим - прошу в гости посмотреть презентационный вариант [**PUT**](https://github.com/MikhailTryakhov/azure-quickstart-templates/blob/master/IPv6/load_balancer_ipv6_demo_put.json) и [**GET**](https://github.com/MikhailTryakhov/azure-quickstart-templates/blob/master/IPv6/load_balancer_ipv6_demo_get.json) запросов.
Восстановить их полностью вы легко сможете, выполнив действия, согласно документации по любой из платформ, например моя родная [**Azure CLI**](https://azure.microsoft.com/ru-ru/documentation/articles/load-balancer-ipv6-internet-cli/).

Обратим внимание на следующие моменты.

**Frontend IP Configuration-Public IP (VIP)**
Если конфигурация Frontend IP ссылается на публичный (public) IP-адрес с версией ipVersion = IPv6, то будут выполнены следующие проверки:
1. Должна существовать другая конфигурация Frontend IP, реализующая версию IPv4.
2. Subnet (подсеть с приватным frontend-адресом для балансировщика) не может быть указана, поскольку в случае IPv6 внутренний балансировщик нагрузки (internal load balancer, ILB) не поддерживается. Следовательно, все ассоциированные данные (приватный IP-адрес, allocation method) будут пустыми (null/not allowed). 
3. Только один виртуальный IP-адрес (VIP) версии IPv6 может быть создан в рамках балансировщика нагрузки.

** Backend Address Pools (Backend) **
Валидация будет состоять в проверке настроек интерфейса сети (NIC). Каждый экземпляр backend adress pool-a может ссылаться на NIC лишь с одним типом поддерживаемых адресов - либо IPv4, либо IPv6.

** Load Balancer Rule (FE <-> BE) **
Если конфигурация Frontend IP ссылается на публичный (public) IP-адрес с версией ipVersion = IPv6, имеют место следующие ограничения:
1. Backend адреса ссылаются на IPv6 конфигурацию NIC.
2. Probe балансировщика нагрузки не поддерживается для версии IPv6

** Inbound NAT pools ** не могут содержать конфигурацию, ссылающуюся на IPv6 VIP.

** Inbound NAT rules ** должны удовлетворять условию, что конфигурации Frontend IP и Backend IP должны быть версии IPv6.
	
** Outbound NAT Rules **
1. Могут содержать правила, основанные на версии IPv6, при условии, что на виртуальный IP-адрес этой версии ссылается конфигуратор frontend IP.
2. В этом случае backend-конфигуратор также должен основываться на IPv6.
3. В рамках балансировщика (load balancer) допустим лишь один виртуальный IP-адрес версии IPv6 .

Ну и, как говорилось выше, в существующей конфигурации невозможно обновить версию IP-адресов (IPv4->IPv6). 

### Настройка Network interface 

Здесь я также предлагаю обратиться непосредственно к примеру конфигурации интерфейса сети 
```json
{
    "name": "mynic1",    
    "id": "/subscriptions/{guid}/resourceGroups/myrg1/providers/Microsoft.Network/networkInterfaces/vm1mynic1",
    "location": "West US",
    "tags": { “key”: “value” } ,
   “etag”: “W/\”00000000-0000-0000-0000-000000000000\”“,
 
    "properties": { 
        "provisioningState": “Updating|Deleting|Failed|Succeeded”,
        "virtualMachine": {"id": "/subscriptions/{guid}/../Microsoft.Compute/virtualMachines/vm1"},
        "macAddress": "BC-31-5B-E2-EE-B1"
        "networkSecurityGroup":{"id":"/subscriptions/{guid}/../Microsoft.Network/networkSecurityGroups/myNSG1" },
        "ipConfigurations": [
{
            "name":  "myIPv4IP1",
 
            "properties": { 
" privateIpAddressVersion ": IPv4 
                "subnet": {"id": "/subscriptions/{guid}/../Microsoft.Network/virtualNetworks/myvnet1/subnets/mysub1"},
                "privateIPAddress":  "10.0.0.8",
                "privateIPAllocationMethod": "Static | Dynamic",
                "publicIPAddress": {}
                "loadBalancerBackendAddressPools": [    
                    {"id": "/subscriptions/{guid}/../Microsoft.Network/loadBalancers/mylb1/backendAddressPools/IPv4BackendPool1"}
                ],
                "loadBalancerInboundNatRules": [
                ]
            }
        }, 
        {
            "name":  "myIpV6Ip1",
            "id": "/subscriptions/{guid}/../Microsoft.Network/networkInterfaces/vm1mynic1/ipConfigurations/ myIpV6Ip1",
            “etag”: “W/\”00000000-0000-0000-0000-000000000000\”“,
            "properties": { 
                "provisioningState": “Updating|Deleting|Failed|Succeeded”,
                " privateIpAddressVersion ": IPv6,    
                "subnet":
                "privateIPAddress  ":  
                "privateIPAllocationMethod": Dynamic",
                "publicIPAddress": 
                "loadBalancerBackendAddressPools": [    
                    {"id": "/subscriptions/{guid}/../Microsoft.Network/loadBalancers/myIPv6lb1/backendAddressPools/IPv6BackendPool1"}
                ],
                "loadBalancerInboundNatRules": []
            }
        }
        ],
        "dnsSettings":   
        {
            "dnsServers": ["1.0.0.1","2.0.0.2"],
            "appliedDnsServers": ["1.0.0.1","2.0.0.2", “3.0.0.3”]
        },
       “enableIPForwarding”: false
    }
}
```

Выбор версии IPv6 накладывает следующие ограничения:
1. NIC должен быть новым, или не привязанным к уже существующей виртуальной машине.
2. Набор конфигураций интерфейса сети должен иметь хотя бы один с версией ipVersion = IPv4.
3. Выбранный NIC - единственный у виртуальной машины с версией ipVersion = IPv6. Multi-NIC здесь на данный момент отсутствует.
4. privateIPAllocationMethod=Dynamic - единственное допустимое значение.
5. Закрытый IP-адрес недоступен в IPv6.
6. Невозможно обновить существующую виртуальную машину на поддержку IPv6 вместо IPv4.

Надеюсь, приведенные сведения помогут Вам успешно преодолеть все сложности, которые могут возникнуть при начале работы с IPv6 в Azure.

### Взгляд в будущее
Если говорить о будущем - на данный момент, увы, я не могу сказать, когда будут произведены обновления, исправляющие весьма кусачие ограничения и когда эта статья станет бесполезной. Пока нет информации ни от команды Azure DNS, ни от NRP (Network Resource Provider). Будет ли возможность более ласковой миграции существующих инфраструктур? Для меня эти вопросы также открыты.

Если говорить о том, что я все же знаю:
* Всё так или иначе движется к полному паритету в поддержке IPv4, IPv6 и IPv4+IPv6.
* Сети и подсети начнут поддерживать IPv6 - идет жаркая работа, и при первой возможности я этими радостями с Вами поделюсь.

Спасибо за внимание, обращайтесь по возникшим вопросам.