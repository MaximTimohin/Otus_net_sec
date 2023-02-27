#### ARP-spoofer.
###  




##### Цель:
* Начать атаку
* Посмотреть результаты


###### Конфигурация компьютера Хакера.
- Посмотрим текущие данные компьютера Hacker

<details>
<summary>Network</summary>

![alt-текст](/lab-6/img/HACKER_network.png)
</summary>

</details>

<details>
<summary>Forward</summary>

![alt-текст](/lab-6/img/FORWARD.png)
</summary>

</details>

###### Конфигурация компьютера атакуемого(Client1).

- Посмотрим текущие данные компьютера Client1

<details>
<summary>Network</summary>


![alt-текст](/lab-6/img/CLIENT1-no-atack.png)
</summary>

</details>


#### Атака на CLIENT1
- Начинаем атаку на Client1

<details>
<summary>Start arp_spoof.py</summary>

![alt-текст](/lab-6/img/SPOOF.png)
</summary>

</details>

- Скрипт удачно запустился

<details>
<summary>tdpdump ARP</summary>

![alt-текст](/lab-6/img/TCPDUMP-ARP.png)
</summary>

</details>

- Через tcpdump на компьютере атаующего убедимся то пакеты пошли

<details>
<summary>Client1_ARP_updated</summary>

![alt-текст](/lab-6/img/CLIENT1-atack.png)
</summary>

</details>

- На компьютере жертвы MAC адрес шлюза поменялся

<details>
<summary>WireShark</summary>

![alt-текст](/lab-6/img/HACKER_ws.png)
</summary>

</details>

- Запустим Wireshark на компьютере атакующего  убедимся, что весь трафик жертвы идёт через компьютер атакующего.

##### Выводы
- Анализ пакетов в Wireshark  показал что атака прошла успешно и трафик жертвы под контролем атакующего.
