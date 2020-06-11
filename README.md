# Архитектура сетей

## Задачи

Построить следующую архитектуру

Сеть office1
- 192.168.2.0/26 - dev
- 192.168.2.64/26 - test servers
- 192.168.2.128/26 - managers
- 192.168.2.192/26 - office hardware

Сеть office2
- 192.168.1.0/25 - dev
- 192.168.1.128/26 - test servers
- 192.168.1.192/26 - office hardware


Сеть central
- 192.168.0.0/28 - directors
- 192.168.0.32/28 - office hardware
- 192.168.0.64/26 - wifi

```
Office1 ---\
-----> Central --IRouter --> internet
Office2----/
```
Итого должны получится следующие сервера
- inetRouter
- centralRouter
- office1Router
- office2Router
- centralServer
- office1Server
- office2Server

### Теоретическая часть
- Найти свободные подсети
- Посчитать сколько узлов в каждой подсети, включая свободные
- Указать broadcast адрес для каждой подсети
- проверить нет ли ошибок при разбиении

### Практическая часть
- Соединить офисы в сеть согласно схеме и настроить роутинг
- Все сервера и роутеры должны ходить в инет черз inetRouter
- Все сервера должны видеть друг друга
- у всех новых серверов отключить дефолт на нат (eth0), который вагрант поднимает для связи
- при нехватке сетевых интервейсов добавить по несколько адресов на интерфейс


## Выполнение

На основе словесного описания задачи создаю схему сетей:
![OTUS-2020-01_Network.svg](./OTUS-2020-01_Network.svg)OTUS-2020-01_Network.svg


### Теоретическая часть

Для расчета количества хостов в сетях использовал формулу, приведенную по ссылке 
<http://infocisco.ru/cisco_formula_subnetting.html>


#### Сеть office1

* 192.168.2.0/26	хостов: 62	broadcast: 192.168.2.63
* 192.168.2.64/26	хостов: 62	broadcast: 192.168.2.127
* 192.168.2.128/26	хостов: 62	broadcast: 192.168.2.191
* 192.168.2.192/26	хостов: 62	broadcast: 192.168.2.255

##### Свободных подсетей нет


#### Сеть office2

* 192.168.1.0/25	хостов: 126	broadcast: 192.168.1.127
* 192.168.1.128/26 	хостов: 62	broadcast: 192.168.1.191
* 192.168.1.192/26 	хостов: 62	broadcast: 192.168.1.255

##### Свободных подсетей нет


#### Сеть central

* 192.168.0.0/28	хостов: 14	broadcast: 192.168.0.15
* 192.168.0.32/28	хостов: 14	broadcast: 192.168.0.47
* 192.168.0.64/26	хостов: 62	broadcast: 192.168.0.127
* 192.168.255.0/30	хостов: 2	broadcast: 192.168.255.3	Сеть для присоединения centralRouter к intetRouter
* 192.168.100.0/30	хостов: 2	broadcast: 192.168.100.3	Сеть для присоединения сети office2
* 192.168.200.0/30	хостов: 2	broadcast: 192.168.200.3	Сеть для присоединения сети office1


##### Свободные подсети:

192.168.0.48/28

192.168.0.128/25

192.168.255.4/30

192.168.255.8/29

192.168.255.16/28

192.168.255.32/27

192.168.255.64/26

192.168.255.128/25


192.168.100.4/30

192.168.100.8/29

192.168.100.16/28

192.168.100.32/27

192.168.100.64/26

192.168.100.128/25


192.168.200.4/30

192.168.200.8/29

192.168.200.16/28

192.168.200.32/27

192.168.200.64/26

192.168.200.128/25


Ошибок при выделении сетей не выявил.



### Практическая часть

* Прописал все недостающие хосты в Vagrantfile, а также настройки для них в соответствие отрисованной схеме.
* У всех серверов default маршурт заменен на новый.


#### Проверка

* office1Server

```
[root@office1Server ~]# ping 192.168.1.194 -c 3
PING 192.168.1.194 (192.168.1.194) 56(84) bytes of data.
64 bytes from 192.168.1.194: icmp_seq=1 ttl=61 time=1.82 ms
64 bytes from 192.168.1.194: icmp_seq=2 ttl=61 time=1.00 ms
64 bytes from 192.168.1.194: icmp_seq=3 ttl=61 time=1.11 ms

--- 192.168.1.194 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.000/1.313/1.821/0.363 ms


[root@office1Server ~]# traceroute 192.168.1.194
traceroute to 192.168.1.194 (192.168.1.194), 30 hops max, 60 byte packets
 1  gateway (192.168.2.193)  0.186 ms  0.100 ms  0.143 ms
 2  192.168.100.1 (192.168.100.1)  0.252 ms  0.234 ms  0.248 ms
 3  192.168.200.2 (192.168.200.2)  0.435 ms  0.474 ms  0.466 ms
 4  192.168.1.194 (192.168.1.194)  0.772 ms  0.708 ms  0.593 ms

 
[root@office1Server ~]# ping ya.ru -c 3
PING ya.ru (87.250.250.242) 56(84) bytes of data.
64 bytes from ya.ru (87.250.250.242): icmp_seq=1 ttl=49 time=9.13 ms
64 bytes from ya.ru (87.250.250.242): icmp_seq=2 ttl=49 time=9.77 ms
64 bytes from ya.ru (87.250.250.242): icmp_seq=3 ttl=49 time=8.49 ms

--- ya.ru ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 8.492/9.133/9.777/0.524 ms


[root@office1Server ~]# traceroute -I ya.ru
traceroute to ya.ru (87.250.250.242), 30 hops max, 60 byte packets
 1  gateway (192.168.2.193)  0.433 ms  0.377 ms  0.346 ms
 2  192.168.100.1 (192.168.100.1)  0.733 ms  0.712 ms  0.683 ms
 3  192.168.255.1 (192.168.255.1)  0.832 ms  0.801 ms  0.995 ms
 4  * * *
 5  * * *
 6  172.16.253.50 (172.16.253.50)  3.769 ms  7.048 ms  7.163 ms
 7  172.16.253.244 (172.16.253.244)  7.524 ms  3.715 ms  3.929 ms
 8  atlant.asr.intelsc.net (188.191.160.129)  3.857 ms  3.666 ms  4.088 ms
 9  77.94.162.185 (77.94.162.185)  3.995 ms  3.929 ms  4.258 ms
10  ae1-atlant-mmts9-msk.naukanet.ru (77.94.160.53)  5.249 ms  5.222 ms  5.171 ms
11  styri.yndx.net (195.208.208.116)  5.701 ms  5.634 ms  4.568 ms
12  ya.ru (87.250.250.242)  9.149 ms  9.415 ms  9.806 ms
```


* office2Server

```
[root@office2Server ~]# ping 192.168.2.194 -c 3
PING 192.168.2.194 (192.168.2.194) 56(84) bytes of data.
64 bytes from 192.168.2.194: icmp_seq=1 ttl=61 time=1.24 ms
64 bytes from 192.168.2.194: icmp_seq=2 ttl=61 time=1.05 ms
64 bytes from 192.168.2.194: icmp_seq=3 ttl=61 time=1.24 ms

--- 192.168.2.194 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 1.059/1.184/1.247/0.092 ms


[root@office2Server ~]# traceroute -I 192.168.2.194
traceroute to 192.168.2.194 (192.168.2.194), 30 hops max, 60 byte packets
 1  gateway (192.168.1.193)  0.261 ms  0.158 ms  0.102 ms
 2  192.168.200.1 (192.168.200.1)  0.742 ms  0.717 ms  0.653 ms
 3  192.168.100.2 (192.168.100.2)  0.716 ms  0.685 ms  0.948 ms
 4  192.168.2.194 (192.168.2.194)  0.920 ms  1.033 ms  1.447 ms
 
 
[root@office2Server ~]# ping 192.168.0.2 -c3
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=62 time=0.809 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=62 time=0.707 ms
64 bytes from 192.168.0.2: icmp_seq=3 ttl=62 time=0.877 ms

--- 192.168.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 0.707/0.797/0.877/0.077 ms
 
 
[root@office2Server ~]# ping ya.ru -c 3
PING ya.ru (87.250.250.242) 56(84) bytes of data.
64 bytes from ya.ru (87.250.250.242): icmp_seq=1 ttl=49 time=19.0 ms
64 bytes from ya.ru (87.250.250.242): icmp_seq=2 ttl=49 time=8.83 ms
64 bytes from ya.ru (87.250.250.242): icmp_seq=3 ttl=49 time=8.43 ms

--- ya.ru ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 8.439/12.102/19.032/4.902 ms


[root@office2Server ~]# traceroute -I ya.ru
traceroute to ya.ru (87.250.250.242), 30 hops max, 60 byte packets
 1  gateway (192.168.1.193)  0.260 ms  0.160 ms  0.122 ms
 2  192.168.200.1 (192.168.200.1)  1.106 ms  1.072 ms  0.998 ms
 3  192.168.255.1 (192.168.255.1)  0.959 ms  1.552 ms  1.533 ms
 4  * * *
 5  * * *
 6  172.16.253.50 (172.16.253.50)  4.531 ms  3.491 ms  3.377 ms
 7  172.16.253.244 (172.16.253.244)  3.583 ms  3.551 ms  3.469 ms
 8  atlant.asr.intelsc.net (188.191.160.129)  3.621 ms  3.811 ms  3.993 ms
 9  77.94.162.185 (77.94.162.185)  4.484 ms  4.978 ms  4.938 ms
10  ae1-atlant-mmts9-msk.naukanet.ru (77.94.160.53)  5.344 ms  5.451 ms  5.658 ms
11  styri.yndx.net (195.208.208.116)  5.613 ms  5.839 ms  4.457 ms
12  ya.ru (87.250.250.242)  9.751 ms  9.242 ms  9.127 ms
```


* centralServer

```
[root@centralServer ~]# ping 192.168.1.194 -c 3
PING 192.168.1.194 (192.168.1.194) 56(84) bytes of data.
64 bytes from 192.168.1.194: icmp_seq=1 ttl=62 time=1.15 ms
64 bytes from 192.168.1.194: icmp_seq=2 ttl=62 time=1.04 ms
64 bytes from 192.168.1.194: icmp_seq=3 ttl=62 time=0.777 ms

--- 192.168.1.194 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.777/0.993/1.154/0.160 ms


[root@centralServer ~]# traceroute -I 192.168.1.194
traceroute to 192.168.1.194 (192.168.1.194), 30 hops max, 60 byte packets
 1  gateway (192.168.0.1)  0.233 ms  0.174 ms  0.141 ms
 2  192.168.200.2 (192.168.200.2)  0.360 ms  0.423 ms  0.393 ms
 3  192.168.1.194 (192.168.1.194)  1.113 ms  1.079 ms  1.047 ms

 
[root@centralServer ~]# ping 192.168.2.194 -c 3
PING 192.168.2.194 (192.168.2.194) 56(84) bytes of data.
64 bytes from 192.168.2.194: icmp_seq=1 ttl=62 time=0.782 ms
64 bytes from 192.168.2.194: icmp_seq=2 ttl=62 time=0.841 ms
64 bytes from 192.168.2.194: icmp_seq=3 ttl=62 time=0.662 ms

--- 192.168.2.194 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 0.662/0.761/0.841/0.081 ms


[root@centralServer ~]# traceroute -I 192.168.2.194
traceroute to 192.168.2.194 (192.168.2.194), 30 hops max, 60 byte packets
 1  gateway (192.168.0.1)  0.259 ms  0.163 ms  0.125 ms
 2  192.168.100.2 (192.168.100.2)  0.447 ms  0.502 ms  0.439 ms
 3  192.168.2.194 (192.168.2.194)  0.727 ms  0.801 ms  0.775 ms

 
[root@centralServer ~]# ping ya.ru -c3
PING ya.ru (87.250.250.242) 56(84) bytes of data.
64 bytes from ya.ru (87.250.250.242): icmp_seq=1 ttl=50 time=8.62 ms
64 bytes from ya.ru (87.250.250.242): icmp_seq=2 ttl=50 time=8.32 ms
64 bytes from ya.ru (87.250.250.242): icmp_seq=3 ttl=50 time=8.28 ms

--- ya.ru ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 8.288/8.413/8.623/0.166 ms


[root@centralServer ~]# traceroute -I ya.ru
traceroute to ya.ru (87.250.250.242), 30 hops max, 60 byte packets
 1  gateway (192.168.0.1)  0.252 ms  0.148 ms  0.110 ms
 2  192.168.255.1 (192.168.255.1)  1.071 ms  1.047 ms  0.974 ms
 3  * * *
 4  * * *
 5  172.16.253.50 (172.16.253.50)  4.351 ms  4.485 ms  4.460 ms
 6  172.16.253.244 (172.16.253.244)  4.426 ms  3.534 ms  3.361 ms
 7  atlant.asr.intelsc.net (188.191.160.129)  3.340 ms  3.589 ms  3.520 ms
 8  77.94.162.185 (77.94.162.185)  3.475 ms  3.621 ms  3.714 ms
 9  ae1-atlant-mmts9-msk.naukanet.ru (77.94.160.53)  4.645 ms  4.614 ms  5.271 ms
10  styri.yndx.net (195.208.208.116)  5.221 ms  5.771 ms  5.345 ms
11  ya.ru (87.250.250.242)  10.063 ms  10.022 ms  10.155 ms
```

По результатам проверки видно, что сервера доступны друг для друга, трафик к внешним ресурсам проходит через 192.168.255.1 (interRouter).

