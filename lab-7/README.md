#### Suricata.
###  




##### Цель:
* Установка и настройка
* Выявление нежелательного трафика


###### Тестовый стенд:
- Маршрутизатор Mikrotik
- Сервер с установленной Suricata + TZSP

<details>
<summary>Характеристики сервера Suricata</summary>

![alt-текст](/lab-7/img/suricata_data.png)
</summary>

</details>

<details>
<summary>Логи Suricata</summary>

![alt-текст](/lab-7/img/suricata_log.png)
</summary>
</details>


##### Выводы
Установлено приложение Suricata и настроена его связь с роутером Mikroitk.
По логам мы можем отслеживать нежелательный трафик.
Далее предстоит отладка данной связки для отключения некоторых правил и подключение Mikrocata для автоматической блокировки на роутере вредоносного трафика.