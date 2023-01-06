#### Управление и мониторинг сетевых устройств.
###  




##### Цель:
* Реализовать безопасное управление и мониторинг сетевых устройств
 - Натроим службы Syslog, NTP, SNMP



#### Схема сети:

  ![alt-текст](/lab-2/img/map-2.png)

  #### Схема коммутации и IP
  <details>
  <summary>Таблица</summary>

  |Device| Interface|IP|comment|
  |:----:|:------|:-----|:--------|
  |R-1|Loopback0|1.1.1.1/24||
  |R-1|Ethernet0/1|192.168.1.1/24|link SW-1_e0/0|
  |R-1|Ethernet0/0|10.1.1.1/30|link R-2_e0/2|
  |R-1|Ethernet0/2|10.1.2.1/24|link Net_Cloud|
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
  <summary>Настройка Syslog, NTP, SNMP</summary>

``
Ниже приведена часть настроек R-1, полные настройки можно посмотреть в файле конфигукации. На остальных устройствах сделаны аналогичные настройки.
``

[файлы конфигурации](./conf/README.md)

``
SNMP
``
```
snmp-server community OtusLabRO RO
snmp-server host 7.7.7.7 version 2c OtusLabRO
```
``
Syslog
``
```
snmp-server community OtusLabRO RO
snmp-server host 7.7.7.7 version 2c OtusLabRO
```
``
NTP
``
```
ntp authentication-key 1 md5 06091B345F42081B 7
ntp authenticate
ntp server 7.7.7.7
```

</details>
</summary>

<details>
<summary>Настройка R-1</summary>

#### Ниже приведена часть настроек R-1, полные настройки можно посмотреть в файле конфигукации.
``
Минимальная длинна пароля сознатнельно сделана 6 знаков т.к. в домашней работе стоит задача настроить сам функцонал.
``

[файлы конфигурации](./conf/README.md)

```
service password-encryption
hostname R-1
security authentication failure rate 10 log
security passwords min-length 6
enable secret 9 $9$AmBka6PnVqs/tq$AlLlk.nt7XkoaAgIMnH8861aOFXiBiP6QPstKu1Hepc
no ip domain lookup
ip domain name otuslab.ru
username user01 secret 9 $9$heaj04nf2ZQzna$yip57i38QXr6NjkXGuKXvp50bVo.mg4q0zvwtcVIGL.
username admin privilege 15 secret 9 $9$ay0V5s1VhXDw7a$Fo4Lr019/Y/mGp/Jm5bEW0VRrlaROJtRwCIUIyBCoHs

```
</details>
</summary>

<details>
<summary>Настройка R-3</summary>

#### Ниже приведена часть настроек R-3, полные настройки можно посмотреть в файле конфигукации.
``
Минимальная длинна пароля сознатнельно сделана 6 знаков т.к. в домашней работе стоит задача настроить сам функцонал.
``

[файлы конфигурации](./conf/README.md)

```
auto secure

```
</details>
</summary>

### Проверка результатов настройки:

<details>
<summary>Подключаемся по ssh к R-3</summary

``
Подключенаемся с syslog сервера
``
```
# ssh -l Maxim 3.3.3.3 -oKexAlgorithms=+diffie-hellman-group1-sha1
The authenticity of host '3.3.3.3 (3.3.3.3)' can't be established.
RSA key fingerprint is SHA256:i6G4xzCWL/r6MJiicgqL2jajiv3ryWbXXJbkFkbMKiA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '3.3.3.3' (RSA) to the list of known hosts.
Password:
 LOGIN or LEAVE, ACCESS is DENIED!!!
R-3>en
Password:
R-3#show runn
Building configuration...

Current configuration : 3611 bytes
!
! No configuration change since last restart
!
version 15.4
no service pad
service tcp-keepalives-in
service tcp-keepalives-out
service timestamps debug datetime msec localtime show-timezone
service timestamps log datetime msec localtime show-timezone
service password-encryption
service sequence-numbers
!
hostname R-3
!
boot-start-marker
boot-end-marker
!
!
security authentication failure rate 10 log
security passwords min-length 6
logging console critical
logging monitor errors
enable secret 5 $1$xIYo$N42BjPBsB6NrpfKL8jLe3/
enable password 7 0229104E182A0E237E5B
!
aaa new-model
!
!
aaa authentication login local_auth local
!
!

R-3#Connection to 3.3.3.3 closed by remote host.
Connection to 3.3.3.3 closed.

```


</details>
</summary>

<details>
<summary>Проверка работы NTP Syslog SNMP на примере R-1</summary

``
NTP
``
```
R-1#show ntp status
Clock is synchronized, stratum 3, reference is 7.7.7.7
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**10
ntp uptime is 781200 (1/100 of seconds), resolution is 4000
reference time is E762846F.E147B080 (14:28:47.880 MSK Fri Jan 6 2023)
clock offset is -3.9638 msec, root delay is 6.44 msec
root dispersion is 54.81 msec, peer dispersion is 3.69 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is -0.000000033 s/s

```
``
Syslog проверим логи на удалённом сервере 7.7.7.7
``
```
# tail -f /data/syslog/1.1.1.1/2023_01_syslog.log
Jan  6 13:24:18 one.one.one.one 54: 000051: Jan  6 10:24:17: %PARSER-5-CFGLOG_LOGGEDCMD: User:console  logged command:ip ssh authentication-retries 2
Jan  6 13:24:18 one.one.one.one 55: 000052: Jan  6 10:24:17: %PARSER-5-CFGLOG_LOGGEDCMD: User:console  logged command:ip ssh logging events
Jan  6 13:24:20 one.one.one.one 56: 000053: Jan  6 10:24:19: %PARSER-5-CFGLOG_LOGGEDCMD: User:console  logged command:ip ssh version 2
Jan  6 13:24:40 one.one.one.one 57: 000054: Jan  6 10:24:39: %SYS-5-CONFIG_I: Configured from console by console
Jan  6 13:36:07 one.one.one.one 58: 000055: Jan  6 10:36:06: %PARSER-5-CFGLOG_LOGGEDCMD: User:console  logged command:!exec: enable
Jan  6 14:20:39 one.one.one.one 59: 000056: Jan  6 11:20:38: %PARSER-5-CFGLOG_LOGGEDCMD: User:console  logged command:!exec: enable
Jan  6 14:20:51 one.one.one.one 60: 000057: Jan  6 11:20:50: %PARSER-5-CFGLOG_LOGGEDCMD: User:console  logged command:snmp-server community * ro
Jan  6 14:20:51 one.one.one.one 61: 000058: Jan  6 11:20:50: %PARSER-5-CFGLOG_LOGGEDCMD: User:console  logged command:snmp-server host 7.7.7.7 version 2c *
Jan  6 14:20:54 one.one.one.one 62: 000059: Jan  6 11:20:53: %SYS-5-CONFIG_I: Configured from console by console
Jan  6 14:39:02 one.one.one.one 63: 000060: Jan  6 11:39:01: %PARSER-5-CFGLOG_LOGGEDCMD: User:console  logged command:!exec: enable

```
``
SNMP
``
```
# /usr/bin/snmpwalk -v2c -c OtusLabRO 1.1.1.1
iso.3.6.1.2.1.1.1.0 = STRING: "Cisco IOS Software, Linux Software (I86BI_LINUX-ADVENTERPRISEK9-M), Version 15.4(2)T4, DEVELOPMENT TEST SOFTWARE
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2015 by Cisco Systems, Inc.
Compiled Thu 08-Oct-15 21:21 by prod_re+"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.9.1.1
iso.3.6.1.2.1.1.3.0 = Timeticks: (797708) 2:12:57.08
iso.3.6.1.2.1.1.4.0 = ""
iso.3.6.1.2.1.1.5.0 = STRING: "R-1.otuslab.ru"
```
</details>
</summary>
