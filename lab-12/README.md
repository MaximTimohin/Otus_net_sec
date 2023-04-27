#### межсайтовая IPsec VPN с аутентификацией


<details>
<summary>Конфигурация сети</summary>

  ![alt-текст](/lab-11/img/map.png)
</summary>
</details>
<details>
  <summary>Таблица распределения IP адресов на оборудовании</summary>


|location        | ip |prefix|gateway| comment|
|:-----------:|:--------:|:---------------:|:-------------:|:---:|
|VPC-1-2|10.100.102.21|24|10.100.102.1|Ofis-1|
|VPC-3-1|10.103.101.2|24|10.103.101.1|Ofis-3|



</details>  


###### Описание сети
Сеть предприятия состоит из одного главного(Ofis-3) офиса и двух второстепенных(Ofis-1,Ofis-2). Главный офис подключен одновременно к двум разным провайдерам для обеспечения резервирования доступа в интернет.
Каждый второстепенный офис имеет два VPN-IPSEC подключения к головному офису.
Компания передаёт конфиденциальную информацию между офиса внутри VPN тоннелей, таким образом защищая её от возможного перехвата во внешней сети.

<details>
<summary>Настройка IPSEC-VPN-FGW1</summary>

```
FortiGate-VM64-KVM # show vpn ipsec phase1-interface FGW1
config vpn ipsec phase1-interface
    edit "FGW1"
        set interface "port3"
        set ike-version 2
        set peertype any
        set net-device disable
        set proposal des-sha256
        set dpd disable
        set comments "VPN: FGW1 (Created by VPN wizard)"
        set dhgrp 21 19 14
        set nattraversal disable
        set remote-gw 100.1.101.2
        set psksecret ENC VyFst1aoE2KpsqSR8HHecoz0tCetYziC11Nx/HHHcGQJYSh/h3pH9p/xga3NZnijBeLc0APHvpQwmoCGFV6+GwkQiLA/dfQVrazk/GTJl4ATpCWQj/sM/o7wcNUJoe3HByokMRMg9J4mmAwoJHH8PQlTI8EKoizlkC7oDezgt7A0Vn3DIrlMXBnAusDdnH1fB8GICw==
    next
end

FortiGate-VM64-KVM # show vpn ipsec phase2-interface FGW1
config vpn ipsec phase2-interface
    edit "FGW1"
        set phase1name "FGW1"
        set proposal des-sha256
        set dhgrp 21 19 14
        set comments "VPN: FGW1 (Created by VPN wizard)"
    next
end

FortiGate-VM64-KVM # show system interface FGW1
config system interface
    edit "FGW1"
        set vdom "root"
        set ip 10.255.255.2 255.255.255.255
        set allowaccess ping
        set type tunnel
        set remote-ip 10.255.255.1 255.255.255.252
        set snmp-index 10
        config ipv6
            set ip6-send-adv enable
            set ip6-other-flag enable
        end
        set interface "port3"
    next
end

```
</summary>
</details>
<details>
<summary>Настройка IPSEC-VPN-FGW1_2</summary>

```
FortiGate-VM64-KVM # show vpn ipsec phase1-interface FGW1_2
config vpn ipsec phase1-interface
    edit "FGW1_2"
        set interface "port4"
        set ike-version 2
        set peertype any
        set net-device disable
        set proposal des-sha256
        set dpd disable
        set comments "VPN: FGW1_2 (Created by VPN wizard)"
        set dhgrp 21 19 14
        set nattraversal disable
        set remote-gw 100.1.101.2
        set psksecret ENC qsdJLZLYa1M6tWDkSl4IeyAssBp36Lf1zcTBSWykpcw86kUKp7kismf7PHeNoY7H4icKramec85J88qdqoN2iUmW3jNVOdYXjsiq1aVfebQWr8Sr9xNsDZ4e2I0WcYwWS/qE8VfB7YkvfcDfFvzI7WwmLdcqWU7vjvt0hjp4NLXL12PxbIPtQ8pVR/+seLQSsDEzhg==
    next
end

FortiGate-VM64-KVM # show vpn ipsec phase2-interface FGW1_2
config vpn ipsec phase2-interface
    edit "FGW1_2"
        set phase1name "FGW1_2"
        set proposal des-sha256
        set dhgrp 21 19 14
    next
end

FortiGate-VM64-KVM # show system interface FGW1_2
config system interface
    edit "FGW1_2"
        set vdom "root"
        set ip 10.255.254.2 255.255.255.255
        set allowaccess ping
        set type tunnel
        set remote-ip 10.255.254.1 255.255.255.252
        set snmp-index 11
        config ipv6
            set ip6-send-adv enable
            set ip6-other-flag enable
        end
        set interface "port4"
    next
end

```
</summary>
</details>

<details>
<summary>Настройка BGP внутри тоннеля FGW1</summary>

```
FortiGate-VM64-KVM # show router bgp
config router bgp
    set as 1000
    config neighbor
        edit "10.255.255.1"
            set soft-reconfiguration enable
            set interface "FGW1"
            set remote-as 64525
            set update-source "FGW1"
        next
    end

```

</summary>
</details>


<details>
<summary>Настройка BGP внутри тоннеля FGW1_2</summary>

```
FortiGate-VM64-KVM # show router bgp
config router bgp
    set as 1000
    config neighbor
        edit "10.255.254.1"
            set soft-reconfiguration enable
            set interface "FGW1_2"
            set remote-as 64525
            set update-source "FGW1_2"
        next
    end

```

</summary>
</details>

<details>
<summary>Статус тоннеля IPSEC-VPN-FGW1</summary>

```
FortiGate-VM64-KVM # get vpn ipsec tunnel name FGW1

gateway
  name: 'FGW1'
  local-gateway: 100.3.101.2:0 (static)
  remote-gateway: 100.1.101.2:0 (static)
  dpd-link: on
  mode: ike-v2
  interface: 'port3' (5)
  rx  packets: 2312  bytes: 280584  errors: 0
  tx  packets: 2343  bytes: 147430  errors: 3
  dpd: disabled
  selectors
    name: 'FGW1'
    auto-negotiate: disable
    mode: tunnel
    src: 0:0.0.0.0/0.0.0.0:0
    dst: 0:0.0.0.0/0.0.0.0:0
    SA
      lifetime/rekey: 43200/27643
      mtu: 1446
      tx-esp-seq: 248
      replay: enabled
      qat: 0
      inbound
        spi: 6a4b83ce
        enc:     des  6bbfadfc3cd3a985
        auth: sha256  87e362f28887c6fe16b974a4c019460dd7ce4704f40f004b289fe9718c7185d1
      outbound
        spi: d0942fbb
        enc:     des  35f924f7ce289b8c
        auth: sha256  d8cd3fa126a65e241f16be75830efbb37fd64b91f4084fa493899cb293664b4a

```
</summary>

</details>

<details>
<summary>Статус тоннеля IPSEC-VPN-FGW1</summary>

```
FortiGate-VM64-KVM # get vpn ipsec tunnel name FGW1_2

gateway
  name: 'FGW1_2'
  local-gateway: 100.2.101.2:0 (static)
  remote-gateway: 100.1.101.2:0 (static)
  dpd-link: on
  mode: ike-v2
  interface: 'port4' (6)
  rx  packets: 2244  bytes: 269848  errors: 0
  tx  packets: 2232  bytes: 138348  errors: 3
  dpd: disabled
  selectors
    name: 'FGW1_2'
    auto-negotiate: disable
    mode: tunnel
    src: 0:0.0.0.0/0.0.0.0:0
    dst: 0:0.0.0.0/0.0.0.0:0
    SA
      lifetime/rekey: 43200/27573
      mtu: 1446
      tx-esp-seq: 24b
      replay: enabled
      qat: 0
      inbound
        spi: 6a4b83cf
        enc:     des  78318600baa3f0cd
        auth: sha256  009d4554ef32736c449ed30a2880186405d1b3e64cbd9d39847d8c2e2fb4ed40
      outbound
        spi: d0942fbc
        enc:     des  3bb8bed060619dfb
        auth: sha256  5e593dacd4f750ad00ea04b40ea81e2fd2ada04c156fbcd51caf9555cb54ffd0


```
</summary>
</details>

<details>
<summary>Состояние BGP внутри тоннеля FGW1</summary>

```
FortiGate-VM64-KVM # get router info bgp neigh 10.255.255.1
VRF 0 neighbor table:
BGP neighbor is 10.255.255.1, remote AS 64525, local AS 1000, external link
  BGP version 4, remote router ID 100.1.101.2
  BGP state = Established, up for 16:29:58
  Last read 00:00:01, hold time is 180, keepalive interval is 60 seconds
  Configured hold time is 180, keepalive interval is 60 seconds
  Neighbor capabilities:
    Route refresh: advertised and received (old and new)
    Address family IPv4 Unicast: advertised and received
    Address family IPv6 Unicast: advertised and received
  Received 1153 messages, 0 notifications, 0 in queue
  Sent 1146 messages, 0 notifications, 0 in queue
  Route refresh request: received 0, sent 0
  NLRI treated as withdraw: 0
  Minimum time between advertisement runs is 30 seconds
  Update source is FGW1

 For address family: IPv4 Unicast
  BGP table version 8, neighbor version 7
  Index 3, Offset 0, Mask 0x8
  Inbound soft reconfiguration allowed
  Community attribute sent to this neighbor (both)
  10 accepted prefixes, 10 prefixes in rib
  33 announced prefixes

 Connections established 1; dropped 0
Local host: 10.255.255.2, Local port: 20704
Foreign host: 10.255.255.1, Foreign port: 179
Egress interface: 13
Nexthop: 10.255.255.2
Nexthop interface: FGW1
Nexthop global: ::
Nexthop local: ::
BGP connection: non shared network
```

</summary>
</details>

<details>
<summary>Состояние BGP внутри тоннеля FGW1_2</summary>

```
FortiGate-VM64-KVM # get router info bgp neigh 10.255.254.1
VRF 0 neighbor table:
BGP neighbor is 10.255.254.1, remote AS 64525, local AS 1000, external link
  BGP version 4, remote router ID 100.1.101.2
  BGP state = Established, up for 16:28:45
  Last read 00:00:09, hold time is 180, keepalive interval is 60 seconds
  Configured hold time is 180, keepalive interval is 60 seconds
  Neighbor capabilities:
    Route refresh: advertised and received (old and new)
    Address family IPv4 Unicast: advertised and received
    Address family IPv6 Unicast: advertised and received
  Received 1141 messages, 0 notifications, 0 in queue
  Sent 1152 messages, 0 notifications, 0 in queue
  Route refresh request: received 0, sent 0
  NLRI treated as withdraw: 0
  Minimum time between advertisement runs is 30 seconds
  Update source is FGW1_2

 For address family: IPv4 Unicast
  BGP table version 8, neighbor version 7
  Index 4, Offset 0, Mask 0x10
  Inbound soft reconfiguration allowed
  Community attribute sent to this neighbor (both)
  10 accepted prefixes, 10 prefixes in rib
  43 announced prefixes

 Connections established 1; dropped 0
Local host: 10.255.254.2, Local port: 179
Foreign host: 10.255.254.1, Foreign port: 10015
Egress interface: 14
Nexthop: 10.255.254.2
Nexthop interface: FGW1_2
Nexthop global: ::
Nexthop local: ::
BGP connection: non shared network

```

</summary>
</details>

<details>
<summary>Трассировка от VPC-1-2 до VPC-3-1 </summary>

```
VPC-1-2> show ip

NAME        : VPC-1-2[1]
IP/MASK     : 10.100.102.21/24
GATEWAY     : 10.100.102.1
DNS         :
DHCP SERVER : 10.100.102.1
DHCP LEASE  : 159137, 217800/108900/190575
DOMAIN NAME : otuslab.ru
MAC         : 00:50:79:66:68:05
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPC-1-2> ping 10.103.101.2

84 bytes from 10.103.101.2 icmp_seq=1 ttl=60 time=7.694 ms
84 bytes from 10.103.101.2 icmp_seq=2 ttl=60 time=3.764 ms
84 bytes from 10.103.101.2 icmp_seq=3 ttl=60 time=3.917 ms
84 bytes from 10.103.101.2 icmp_seq=4 ttl=60 time=4.982 ms
^C
VPC-1-2> trace 10.103.101.2 -P 1
trace to 10.103.101.2, 8 hops max (ICMP), press Ctrl+C to stop
 1   10.100.102.1   0.428 ms  0.252 ms  0.265 ms              # --- SW-1-2
 2   10.100.22.1   0.780 ms  0.759 ms  0.749 ms               # --- FGW-1
 3   10.255.255.2   2.792 ms  2.091 ms  1.808 ms              # --- VPNip-at-FGW3
 4   10.103.21.2   2.849 ms  2.264 ms  2.783 ms               # --- R-3-1
 5   10.103.101.2   2.641 ms  2.739 ms  2.816 ms              # --- VPC-3-1


```
</summary>
</details>

##### Выводы
Судя по трассировке два клиента из удалённых офисов видят друг друга через VPN
соединение построенными между офисами. Таким образом данные компании не попадают во внешнюю сеть в открытом виде и являются защищёнными от перехвата.
