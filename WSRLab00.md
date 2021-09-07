# WSRLab00

Задание:

1. Вы заказали инстансы (SRV1,SRV2,SRV3) и облачные
маршрутизаторы Cisco у облачного сервиса kp11cloud.

Инстансы находятся в регионах (region1, region2, region3).
Так же у вас есть DNS имя для приложения (mycorp.ru),
которое, ссылается на Load Balancer.

2. Вашей задачей является - настроить VPN между
R1,R2,R3.

3. На SRV1,SRV2,SRV3 создайте одностраничный сайт.

4. На LoadBalancer настройте nginx, так что бы при
обращении клиента к mycorp.ru открывался работающий
инстанс. Приорететы - SRV1, затем SRV2, затем SRV3.

| Region 1              | Region 2              | Region 3              | Cloud services |
| -------------         | -------------         | -------------         | -------------  |
|R1 - 10.10.11.1/24     |R2 - 10.10.12.1/24     |R3 - 10.10.13.1/24     |Load Balancer - 10.31.14.127|
|r1.region1.kp11cloud.ru|r2.region2.kp11cloud.ru|r3.region3.kp11cloud.ru|DNS - 10.31.14.126          |
|GW - 10.10.11.254      |GW - 10.10.11.254      |GW - 10.10.11.254      |Application DNS - mycorp.ru |


![1](https://user-images.githubusercontent.com/79700810/132322342-9c6db933-44b5-4372-a114-7c90af87af79.png)
--
# Базовая Конфигурация R1, R2, R3

| R1              | R2             |R3             | 
| -------------         | -------------         | -------------         |
|en      |en      |en      |
|conf t|conf t|conf t|
|hostname R1      |hostname R2      |hostname R3      |
|interface gigabitethernet 0/0      |interface gigabitethernet 0/0      |interface gigabitethernet 0/0      |
|no shutdown      |no shutdown      |no shutdown      |
|ip address 10.10.11.1 255.255.255.0      |ip address 10.10.12.1 255.255.255.0      |ip address 10.10.13.1 255.255.255.0      |
|exit      |exit     |exit      |
|interface gigabitethernet 0/1      |interface gigabitethernet 0/1    |interface gigabitethernet 0/1      |
|no shutdown      |no shutdown      |no shutdown      |
|ip address 192.168.11.1 255.255.255.0      |ip address 192.168.12.1 255.255.255.0     |ip address 192.168.13.1 255.255.255.0      |
|exit      |exit     |exit      |
|ip route 0.0.0.0 0.0.0.0 10.10.11.254|ip route 0.0.0.0 0.0.0.0 10.10.11.254     |ip route 0.0.0.0 0.0.0.0 10.10.11.254     |

# Настройка GRE R1, R2, R3

