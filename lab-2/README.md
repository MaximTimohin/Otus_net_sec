#### Административные роли.
###  




##### Цель:
* Базовая настройка сетевых устройств.
* Настрояка IP адресов и маршутизации.
* Настройка авторизации команд с использованием уровней привилегий и интерфейса командной строки на основе ролей.



#### Схема сети:

  ![alt-текст](/lab-1/img/map1.png)

  #### Схема коммутации и IP
  <details>
  <summary>Таблица</summary>

  |Device| Interface|IP|comment|
  |:----:|:------|:-----|:--------|
  |R-1|Loopback0|1.1.1.1/24||
  |R-1|Ethernet0/1|192.168.1.1/24|link SW-1_e0/0|
  |R-1|Ethernet0/0|10.1.1.1/30|link R-2_e0/2|
  ||
  |R-2|Loopback0|2.2.2.2/24||
  |R-2|Ethernet0/2|10.1.1.2/30|link R-1_e0/0|
  |R-2|Ethernet0/3|10.2.2.1/30|link R-3_e0/0|  
  ||
  |R-3|Loopback0|3.3.3.3/24||
  |R-3|Ethernet0/0|10.2.2.2/30|link R-2_e0/3|
  |R-3|Ethernet0/1|192.168.3.1/24|link SW-3_e0/0|
  ||
  |SW-1|Vlan 1|192.168.1.2/24||
  |SW-1|Ethernet0/0||link R-1_e0/2|
  |SW-1|Ethernet0/3||link VPC-A|  
  ||
  |SW-3|Vlan 1|192.168.3.2/24||
  |SW-3|Ethernet0/0||link R-3_e0/1|
  |SW-3|Ethernet0/3||link VPC-C|
  ||
  |VPC-A||192.168.1.3/24||
  |VPC-C||192.168.3.3/24||

  </details>



  ### Настройка:

  <details>
  <summary>Базовая настройка</summary>

  #### Ниже приведены часть комманд базовой настройки устройства, полные настройки можно посмотреть в файле конфигурации по ссылке ниже.

[файлы конфигурации](./conf/README.md)

  ```
  ip domain-name otuslab.ru

  hostname R-1
  no ip domain lookup
  no ip http server
  no ip http secure-server

  service password-encryption
  aaa new-model
  banner motd c LOGIN or LEAVE, ACCESS is DENIED!!! c
  security passwords min-length 6
  security authentication failure rate 10 log
  clock timezone MSK 3 0
  ```
  </details>
</summary>

<details>
<summary>Настройка маршрутизации</summary>

#### Настройки IP на интерфейсах можно посмотреть в файлах конфигурации, они соответствуют схеми сети.

[файлы конфигурации](./conf/README.md)

```
R-1

router ospf 1
 router-id 1.1.1.1
 passive-interface Ethernet0/1
 network 1.1.1.0 0.0.0.255 area 0
 network 10.1.1.0 0.0.0.3 area 0
 network 192.168.1.0 0.0.0.255 area 0

R-2

router ospf 1
 router-id 2.2.2.2
 network 2.2.2.0 0.0.0.255 area 0
 network 10.1.1.0 0.0.0.3 area 0
 network 10.2.2.0 0.0.0.3 area 0


R-3

router ospf 1
 router-id 3.3.3.3
 passive-interface Ethernet0/1
 network 3.3.3.0 0.0.0.255 area 0
 network 10.2.2.0 0.0.0.3 area 0
 network 192.168.3.0 0.0.0.255 area 0


```
</details>
</summary>

<details>
<summary>Настройка административных ролей</summary>

#### Настройки приведены для R-1, для R-3 сделаны аналогичные настройки.

[файлы конфигурации](./conf/README.md)

```
parser view admin1
 secret 5 $1$sZVq$8DX1RA8kOOlWrcGstrCef0
 commands exec include all configure terminal
 commands exec include configure
 commands exec include all show
!
parser view admin2
 secret 5 $1$.oXY$Q8W9yCx8Rd9M5eH/rm3Vl/
 commands exec include all show
!
parser view tech
 secret 5 $1$4.//$5rfowVsKo6IRlMB/ib/HA1
 commands exec include show ip interface brief
 commands exec include show ip interface
 commands exec include show ip
 commands exec include show version
 commands exec include show parser view
 commands exec include show parser
 commands exec include show interfaces
 commands exec include show

```
</details>
</summary>

### Проверка результатов настройки:

<details>
<summary>Проверка связанности и маршутизации</summary

``
Ping между узлами VPC-A, VPC-C
``
```
VPC-A> ping 192.168.3.3 -c 4

84 bytes from 192.168.3.3 icmp_seq=1 ttl=61 time=1.572 ms
84 bytes from 192.168.3.3 icmp_seq=2 ttl=61 time=1.438 ms
84 bytes from 192.168.3.3 icmp_seq=3 ttl=61 time=1.829 ms
84 bytes from 192.168.3.3 icmp_seq=4 ttl=61 time=2.012 ms
```
``
Trace между узлами VPC-A, VPC-C
``
```
VPC-A> trace 192.168.3.3 -P 1
trace to 192.168.3.3, 8 hops max (ICMP), press Ctrl+C to stop
 1   192.168.1.1   0.620 ms  0.469 ms  0.470 ms
 2   10.1.1.2   0.903 ms  0.752 ms  0.828 ms
 3   10.2.2.2   1.034 ms  1.001 ms  1.038 ms
 4   192.168.3.3   1.645 ms  1.333 ms  1.376 ms

VPC-A>

```
``
R-1 show ip route ospf
``
```
R-1#             show ip route ospf | beg Gateway
Gateway of last resort is not set

      2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/11] via 10.1.1.2, 04:01:35, Ethernet0/0
      3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/21] via 10.1.1.2, 03:56:26, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
O        10.2.2.0/30 [110/20] via 10.1.1.2, 04:01:35, Ethernet0/0
O     192.168.3.0/24 [110/30] via 10.1.1.2, 03:49:44, Ethernet0/0

```
``
R-2 show ip route ospf
``
```
R-2#show ip route ospf | beg Gateway
Gateway of last resort is not set

      1.0.0.0/32 is subnetted, 1 subnets
O        1.1.1.1 [110/11] via 10.1.1.1, 04:05:33, Ethernet0/2
      3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/11] via 10.2.2.2, 04:01:25, Ethernet0/3
O     192.168.1.0/24 [110/20] via 10.1.1.1, 03:51:48, Ethernet0/2
O     192.168.3.0/24 [110/20] via 10.2.2.2, 03:54:42, Ethernet0/3
R-2#
```
``
R-3 show ip route ospf
``
```
R-3#show ip route ospf | beg Gateway
Gateway of last resort is not set

      1.0.0.0/32 is subnetted, 1 subnets
O        1.1.1.1 [110/21] via 10.2.2.1, 04:00:18, Ethernet0/0
      2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/11] via 10.2.2.1, 04:00:18, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
O        10.1.1.0/30 [110/20] via 10.2.2.1, 04:00:18, Ethernet0/0
O     192.168.1.0/24 [110/30] via 10.2.2.1, 03:50:36, Ethernet0/0
R-3#
```
</details>
</summary>

<details>
<summary>Проверка административных ролей</summary

``
Проверка проводится на R-1
``

```
R-1>en
Password:
R-1#enable view admin1
Password:

R-1#show parser view
Current view is 'admin1'

R-1#?
Exec commands:
  configure   Enter configuration mode
  credential  load the credential info from file system
  do-exec     Mode-independent "do-exec" prefix support
  enable      Turn on privileged commands
  exit        Exit from the EXEC
  show        Show running system information

R-1#show ?
  aaa                       Show AAA values
  access-expression         List access expression
  access-lists              List access lists
  acircuit                  Access circuit info
  adjacency                 Adjacent nodes
  aliases                   Display alias commands
  alps                      Alps information
  appfw                     Application Firewall information
  application               Application Routing
  archive                   Archive functions
  arp                       ARP table
  async                     Information on terminal lines used as router
                            interfaces
  authentication            Shows Auth Manager registrations or sessions
  auto                      Show Automation Template
  backhaul-session-manager  Backhaul Session Manager information
  backup                    Backup status
  banner                    Display banner information
  beep                      Show BEEP information
  bfd                       BFD protocol info
  bgp                       BGP information
  bootvar                   Boot and related environment variable

R-1#


```

```
R-1#enable view admin2
Password:

R-1#show  parser view
Current view is 'admin2'

R-1#?
Exec commands:
  credential  load the credential info from file system
  do-exec     Mode-independent "do-exec" prefix support
  enable      Turn on privileged commands
  exit        Exit from the EXEC
  show        Show running system information

R-1#

```

```
R-1#enable view tech
Password:

R-1#show  parser view
Current view is 'tech'

R-1#?
Exec commands:
  credential  load the credential info from file system
  do-exec     Mode-independent "do-exec" prefix support
  enable      Turn on privileged commands
  exit        Exit from the EXEC
  show        Show running system information

R-1#show ?
  banner      Display banner information
  disk0:      display information about disk0: file system
  disk1:      display information about disk1: file system
  interfaces  Interface status and configuration
  ip          IP information
  parser      Display parser information
  unix:       display information about unix: file system
  version     System hardware and software status

R-1#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.1.1.1        YES manual up                    up
Ethernet0/1                192.168.1.1     YES manual up                    up
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
Loopback0                  1.1.1.1         YES manual up                    up
R-1#show ip route
            ^
% Invalid input detected at '^' marker.

R-1#
```


</details>
</summary>
