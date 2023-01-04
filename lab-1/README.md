#### Административные роли.
###  




##### Цель:
* базовая настройка сетевых устройств.
* Настрояка IP адресов и маршутизации.
* Настройка авторизации команд с использованием уровней привилегий и интерфейса командной строки на основе ролей.



#### Схема сети:

  ![alt-текст](/lab-1/img/map1.png)

  #### Используемы IP сети
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

  ###### Базовая настройка
  <details>
  <summary>config</summary>

  #### Ниже приведены часть комманд базовой настройки устройства, полные настройки можно посмотреть в файле конфигурации по ссылке ниже.

[конфигурации](./lab-1/conf/)

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
