#### Настройка межсетевого экрана "Произвести настройку межсетевого экрана на основе зон"

##### Задача.

- Client1 может обратиться к Client2/Client3 через icmp и http.
- Client 3 может обратиться только к Client2 через icmp.
- Client 2 сам не может обратиться никуда.

<details>
<summary>Конфигурация сети</summary>

  ![alt-текст](/lab-10-1/img/map.png)
</summary>
</details>

<details>
  <summary>Таблица распределения IP адресов на оборудовании</summary>


|device| ip |prefix|gateway|port|comment|
|:-----------:|:--------:|:---------------:|:-------------:|:---:|:---:|
|R1|10.101.255.1|24||e0/0|VPC-1|
|R1|10.101.101.1|24||e0/1|to R2|
|R1|10.103.103.1|24||e0/3|to R3|
|R2|10.102.255.1|24||e0/0|VPC-2|
|R2|10.101.101.2|24||e0/1|to R1|
|R2|10.102.103.2|24||e0/2|to R3|
|R3|10.103.255.1|24||e0/0|VPC-3|
|R3|10.102.103.3|24||e0/2|to R2|
|R3|10.103.103.3|24||e0/3|to R1|
|VPC-1|10.101.255.2|24|10.101.255.1|eth0|to R1|
|VPC-2|10.102.255.2|24|10.102.255.1|eth0|to R2|
|VPC-3|10.103.255.2|24|10.103.255.1|eth0|to R3|

</details>  

</summary>

</details>


<details>
<summary>Общие настройки</summary>

###### На каждом устройстве настроена адресация согласно схеме и обеспечена связанность.
Дополнительно сделаны базовые настройки оборудования и настроен OSPF.

[Конфигурации устройств](./config/)

</summary>
</details>


<details>
<summary>Настройка R1</summary>

```
ip access-list extended LOCAL-TO-INTERNET
 permit tcp 10.101.255.0 0.0.0.255 10.103.255.0 0.0.0.255 eq www
 permit tcp 10.101.255.0 0.0.0.255 10.102.255.0 0.0.0.255 eq www
 permit icmp 10.101.255.0 0.0.0.255 10.103.255.0 0.0.0.255
 permit icmp 10.101.255.0 0.0.0.255 10.102.255.0 0.0.0.255
```

```
class-map type inspect match-all CLASS-ICMP-LOCAL-TO-INTERNET
 match access-group name LOCAL-TO-INTERNET
 match protocol icmp
class-map type inspect match-any CLASS-HTTP-LOCAL-TO-INTERNET
 match protocol http
 match protocol icmp
!
policy-map type inspect LOCAL-TO-INTERNET
 class type inspect CLASS-ICMP-LOCAL-TO-INTERNET
  inspect
 class type inspect CLASS-HTTP-LOCAL-TO-INTERNET
  inspect
 class class-default
  drop log
!
zone security LOCAL
zone security INTERNET
zone-pair security LOCAL_TO_INTERNET source LOCAL destination INTERNET
 service-policy type inspect LOCAL-TO-INTERNET
zone-pair security INTERNET_TO_LOCAL source INTERNET destination LOCAL

```

```
interface Ethernet0/0
 description VPC-1
 ip address 10.101.255.1 255.255.255.0
 zone-member security LOCAL

interface Ethernet0/1
 description R2
 ip address 10.101.101.1 255.255.255.0
 zone-member security INTERNET

interface Ethernet0/3
  description R3
  ip address 10.103.103.1 255.255.255.0
  zone-member security INTERNET

```

</summary>
</details>







<details>
<summary>Настройка R2</summary>


```
ip access-list extended INTERNET-TO-LOCAL-ICMP
 permit icmp 10.101.255.0 0.0.0.255 10.102.255.0 0.0.0.255
 permit icmp 10.103.255.0 0.0.0.255 10.102.255.0 0.0.0.255
ip access-list extended INTERNET-TO-LOCAL-L7
 permit tcp 10.101.255.0 0.0.0.255 10.102.255.0 0.0.0.255 eq www

```

```
class-map type inspect match-all CLASS-ICMP-INTERNET-TO-LOCAL
 match access-group name INTERNET-TO-LOCAL-ICMP
 match protocol icmp
class-map type inspect match-all CLASS-L7-INTERNET-TO-LOCAL
 match access-group name INTERNET-TO-LOCAL-L7
 match protocol http
!
policy-map type inspect INTERNET-TO-LOCAL
 class type inspect CLASS-ICMP-INTERNET-TO-LOCAL
  inspect
 class type inspect CLASS-L7-INTERNET-TO-LOCAL
  inspect
 class class-default
  drop log
!
zone security LOCAL
zone security INTERNET
zone-pair security INTERNET_TO_LOCAL source INTERNET destination LOCAL
 service-policy type inspect INTERNET-TO-LOCAL

```
```

interface Ethernet0/0
 description VPC-2
 ip address 10.102.255.1 255.255.255.0
 zone-member security LOCAL

interface Ethernet0/1
description R1
ip address 10.101.101.2 255.255.255.0
zone-member security INTERNET

interface Ethernet0/2
 description R3
 ip address 10.102.103.2 255.255.255.0
 zone-member security INTERNET


```
</summary>
</details>

<details>
<summary>Настройка R3</summary>

```
ip access-list extended INTERNET-TO-LOCAL
 permit icmp 10.101.255.0 0.0.0.255 10.103.255.0 0.0.0.255
ip access-list extended LOCAL-TO-INTERNET
 permit icmp 10.103.255.0 0.0.0.255 10.102.255.0 0.0.0.255
 deny   ip any any
```

```
class-map type inspect match-all CLASS-ICMP-LOCAL-TO-INTERNET
 match access-group name LOCAL-TO-INTERNET
 match protocol icmp
class-map type inspect match-all INTERNET-TO-LOCAL
 match access-group name INTERNET-TO-LOCAL
 match protocol icmp
!
policy-map type inspect LOCAL-TO-INTERNET
 class type inspect CLASS-ICMP-LOCAL-TO-INTERNET
  inspect
 class type inspect INTERNET-TO-LOCAL
  pass
 class class-default
  drop log
policy-map type inspect INTERNET-TO-LOCAL
 class type inspect INTERNET-TO-LOCAL
  inspect
 class class-default
  drop
!
zone security LOCAL
zone security INTERNET
zone-pair security LOCAL_TO_INTERNET source LOCAL destination INTERNET
 service-policy type inspect LOCAL-TO-INTERNET
zone-pair security INTERNET_TO_LOCAL source INTERNET destination LOCAL
 service-policy type inspect INTERNET-TO-LOCAL

```

```
interface Ethernet0/0
 description VPC-3
 ip address 10.103.255.1 255.255.255.0
 zone-member security LOCAL

interface Ethernet0/2
  description R2
  ip address 10.102.103.3 255.255.255.0
  zone-member security INTERNET

interface Ethernet0/3
   description R1
   ip address 10.103.103.3 255.255.255.0
   zone-member security INTERNET

```

</summary>
</details>


##### Результаты настройки

<details>
<summary>Тестирование доступности с VPC-1</summary>

```
VPC-1> show ip

NAME        : VPC-1[1]
IP/MASK     : 10.101.255.2/24
GATEWAY     : 10.101.255.1
DNS         :
MAC         : 00:50:79:66:68:0c
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPC-1> ping 10.103.255.2 -1 -c 3

84 bytes from 10.103.255.2 icmp_seq=1 ttl=62 time=1.590 ms
84 bytes from 10.103.255.2 icmp_seq=2 ttl=62 time=0.933 ms
84 bytes from 10.103.255.2 icmp_seq=3 ttl=62 time=1.909 ms

VPC-1> ping 10.102.255.2 -1 -c 3

84 bytes from 10.102.255.2 icmp_seq=1 ttl=62 time=1.466 ms
84 bytes from 10.102.255.2 icmp_seq=2 ttl=62 time=1.048 ms
84 bytes from 10.102.255.2 icmp_seq=3 ttl=62 time=2.076 ms

VPC-1>

```
</summary>
</details>


<details>
<summary>Тестирование доступности с VPC-2</summary>

```

VPC-2> show ip

NAME        : VPC-2[1]
IP/MASK     : 10.102.255.2/24
GATEWAY     : 10.102.255.1
DNS         :
MAC         : 00:50:79:66:68:0d
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPC-2> ping 10.101.255.2 -1 -c 3

10.101.255.2 icmp_seq=1 timeout
10.101.255.2 icmp_seq=2 timeout
10.101.255.2 icmp_seq=3 timeout

VPC-2> ping 10.102.255.2 -1 -c 3

10.102.255.2 icmp_seq=1 ttl=64 time=0.001 ms
10.102.255.2 icmp_seq=2 ttl=64 time=0.001 ms
10.102.255.2 icmp_seq=3 ttl=64 time=0.001 ms

VPC-2> ping 10.103.255.2 -1 -c 3

10.103.255.2 icmp_seq=1 timeout
10.103.255.2 icmp_seq=2 timeout
10.103.255.2 icmp_seq=3 timeout

VPC-2>

```

</summary>
</details>


<details>
<summary>Тестирование доступности с VPC-3</summary>

```
VPC-3> show ip

NAME        : VPC-3[1]
IP/MASK     : 10.103.255.2/24
GATEWAY     : 10.103.255.1
DNS         :
MAC         : 00:50:79:66:68:0e
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPC-3> ping 10.102.255.2 -1 -c 3

84 bytes from 10.102.255.2 icmp_seq=1 ttl=62 time=1.289 ms
84 bytes from 10.102.255.2 icmp_seq=2 ttl=62 time=0.896 ms
84 bytes from 10.102.255.2 icmp_seq=3 ttl=62 time=0.967 ms

VPC-3> ping 10.101.255.2 -1 -c 3

10.101.255.2 icmp_seq=1 timeout
10.101.255.2 icmp_seq=2 timeout
10.101.255.2 icmp_seq=3 timeout

VPC-3>
```

</summary>
</details>

##### Выводы
Согласно задаче:
1. VPC-1 пингует VPC-2, VPC-3.
2. VPC-2 пингует только свой шлюз и ни кого более.
3. VPC-3 пингует только VPC-2.

Доспупность http не возможно на VPC.
