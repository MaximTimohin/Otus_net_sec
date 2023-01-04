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
</details>
</summary>

<details>

#### Проверка проводится на R-1

<summary>Проверка административных ролей</summary


``
Проверка проводится на R-1
``

</details>
</summary>
