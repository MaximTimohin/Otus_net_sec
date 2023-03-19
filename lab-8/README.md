#### Обнаружение атаки arp_spoofing и отправка сообщения на почту.
###  


##### Цель:
* Доработка скрипт arp_spoof_detector.py




<details>
<summary>Доработанный скрипт</summary>

```
#!/usr/bin/env python
import scapy.all as scapy
#import smtp
import smtplib                        # Импортируем библиотеку smtplib для отправки e-mail
from email.mime.text import MIMEText  # Импортируем MIMEText, чтобы определить текст сообщения
import getpass                       # Импортируем библиотеку getpass, чтобы скрыть ваш пароль

def get_mac(ip):
    ''' Get mac address by ip '''
    arp_req = scapy.ARP(pdst=ip)
    broadcast = scapy.Ether(dst='ff:ff:ff:ff:ff:ff')
    arp_req_broadcast = broadcast/arp_req
    resp_list = scapy.srp(arp_req_broadcast, timeout=1, verbose=False)[0]

    return resp_list[0][1].hwsrc

def send_alarm(email, password, message):
  ''' Отправляет уведомление об атаке на email с использованием SMTP '''
  mail_server = 'smtp.MAILSERVER'  # Выберите SMTP-сервер своего провайдера электронной почты
  mail_port = 587                 # Порт SMTP-сервера
  recipient_mail = email          # Email-адрес получателя уведомления
  sender_mail = email             # Email-адрес отправителя
  sender_password = password    # Пароль отправителя (включая надежную аутентификацию)
  message = MIMEText(message)     # Определение текста сообщения
  message['Subject'] = 'ALARM! ARP-spoofing attack was detected！'  # Задание темы сообщения
  message['From'] = sender_mail   # Задание адреса отправителя
  message['To'] = recipient_mail  # Задание адреса получателя
  try:
    server = smtplib.SMTP(mail_server, mail_port)  # Подключаемся к SMTP-серверу
    server.starttls()                             # Начать безопасную связь
    server.login(sender_mail, sender_password)     # Входим в учетную запись отправителя
    server.sendmail(sender_mail, recipient_mail, message.as_string()) # Отправляем сообщение
    server.quit()                                 # Выходим из учетной записи отправителя
    print('[Info] E-mail is sent successfully!')
  except Exception as ex:
    print(f'[Error] {ex}')

def process_sniffed_packet(packet):
   if packet.haslayer(scapy.ARP) and packet[scapy.ARP].op == 2:
      try:
         real_mac = get_mac(packet[scapy.ARP].psrc)
         response_mac = packet[scapy.ARP].hwsrc
         if real_mac != response_mac:
         # Отправляем уведомление на почту
           email = 'UserMail'    # Задайте свой адрес электронной почты
           password = getpass.getpass()   # Запрос пароля для отправителя уведомлений
           message = 'ALARM! ARP-spoofing attack was detected! Please check your network.'  # Текстовое сообщение
           send_alarm(email, password, message)  # Отправляем уведомление об атаке
           print('ALARM! ARP-spoofing attack was detected!')
      except IndexError:
         pass



def sniff(interface):
   scapy.sniff(iface=interface, store=False, prn=process_sniffed_packet)

sniff('ens18')

```
</summary>

</details>

<details>
<summary>Запуск скрипта</summary>

##### Запускаем Arpspoof на атакующей машине.

![alt-текст](/lab-8/img/spoof.png)

##### Запускаем arp_spoof_detector на компьютере жертвы.
![alt-текст](/lab-8/img/start.png)
</summary>
</details>

<details>
<summary>Письмо пришло!</summary>

![alt-текст](/lab-8/img/email.png)
</summary>
</details>

##### Выводы
Доработанный скрипт определил атаку и прислал оповещение на почту.
