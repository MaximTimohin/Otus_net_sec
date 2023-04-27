#### Протокол шифрования

##### Цель
* Выбрать и обосновать протокол шифрования в моей сети


<details>
<summary>Конфигурация сети</summary>

  ![alt-текст](/lab-11/img/map.png)
</summary>
</details>


###### Описание сети
Сеть предприятия состоит из одного главного(Ofis-3) офиса и двух второстепенных(Ofis-1,Ofis-2). Главный офис подключен одновременно к двум разным провайдерам для обеспечения резервирования доступа в интернет.
Каждый второстепенный офис имеет два VPN-IPSEC подключения к головному офису.
Компания передаёт конфиденциальную информацию между офиса внутри VPN тоннелей, таким образом защищая её от возможного перехвата во внешнец сети.

###### Обоснование выбора протокола шифрования
IPsec DES 256 использует 256-битный ключ, который обеспечивает высокую степень защиты от взлома и подбора паролей. Этот метод шифрования также обеспечивает конфиденциальность, целостность и аутентификацию данных, что делает его идеальным для использования в корпоративной среде.
Кроме того, IPsec DES 256 является стандартом безопасности для многих организаций. Это означает, что использование этого метода шифрования поможет обеспечить соответствие стандартам безопасности.


<details>
<summary>Настройка VPN</summary>

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

```
</summary>
</details>

<details>
<summary>Статус VPN соединения</summary>

```
gateway
  name: 'FGW1'
  local-gateway: 100.3.101.2:0 (static)
  remote-gateway: 100.1.101.2:0 (static)
  dpd-link: on
  mode: ike-v2
  interface: 'port3' (5)
  rx  packets: 2259  bytes: 274212  errors: 0
  tx  packets: 2290  bytes: 144180  errors: 3
  dpd: disabled
  selectors
    name: 'FGW1'
    auto-negotiate: disable
    mode: tunnel
    src: 0:0.0.0.0/0.0.0.0:0
    dst: 0:0.0.0.0/0.0.0.0:0
    SA
      lifetime/rekey: 43200/29005
      mtu: 1446
      tx-esp-seq: 213
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
