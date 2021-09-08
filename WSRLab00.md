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

| R1              | R2             |R3             | 
| -------------         | -------------         | -------------         |
|interface Tunne 1     |interface Tunne1     |interface Tunne2    |
|ip address 172.16.1.1 255.255.255.0|ip address 172.16.1.2 255.255.255.0|ip address 172.16.2.2 255.255.255.0|
|tunnel mode gre ip|tunnel mode gre ip|tunnel mode gre ip|
|tunnel source gigabitEthernet 0/0|tunnel source gigabitEthernet 0/0|tunnel source gigabitEthernet 0/0|
|tunnel destination 10.10.12.1|tunnel destination 10.10.11.1|tunnel destination 10.10.11.1|
|exit|exit|exit|
|interface Tunne 2     |interface Tunne3     |interface Tunne3     |
|ip address 172.16.2.1 255.255.255.0|ip address 172.16.3.1 255.255.255.0|ip address 172.16.3.2 255.255.255.0|
|tunnel mode gre ip|tunnel mode gre ip|tunnel mode gre ip|
|tunnel source gigabitEthernet 0/0|tunnel source gigabitEthernet 0/0|tunnel source gigabitEthernet 0/0|
|tunnel destination 10.10.13.1|tunnel destination 10.10.13.1|tunnel destination 10.10.12.1|
|exit|exit|exit|


# Настройка BGP R1, R2, R3


| R1              | R2             |R3             | 
| -------------         | -------------         | -------------         |
|router bgp 65000    |router bgp 65001     |router bgp 65002   |
|network 192.168.11.0 mask 255.255.255.0    |network 192.168.12.0 mask 255.255.255.0     |network 192.168.13.0 mask 255.255.255.0   |
|network 172.16.1.0 mask 255.255.255.0    |network 172.16.1.0 mask 255.255.255.0     |network 172.16.1.0 mask 255.255.255.0   |
|network 172.16.2.0 mask 255.255.255.0    |network 172.16.3.0 mask 255.255.255.0     |network 172.16.3.0 mask 255.255.255.0   |
|neighbor 172.16.1.2 remote-as 65001    |neighbor 172.16.1.1 remote-as 65000    |neighbor 172.16.1.1 remote-as 65000   |
|neighbor 172.16.2.2 remote-as 65002   |neighbor 172.16.3.2 remote-as 65002    |neighbor 172.16.2.2 remote-as 65001   |
|exit    |exit    |exit   |

# Настройка ipsec ikev1 R1, R2, R3

| R1              | R2             |R3             | 
| -------------         | -------------         | -------------         |
|crypto isakmp policy 1    |crypto isakmp policy 1     |crypto isakmp policy 1   |
|authentication pre-share    |authentication pre-share     |authentication pre-share   |
|hash md5    |hash md5     |hash md5   |
|encr 3des    |encr 3des     |encr 3des   |
|group 2    |group 2     |group 2   |
|lifetime 86400    |lifetime 86400     |lifetime 86400   |
|!    |!     |!  |
|crypto isakmp key CISCO123 address 10.10.12.1   |crypto isakmp key CISCO123 address 10.10.11.1     |crypto isakmp key CISCO123 address 10.10.11.1   |
|crypto isakmp key CISCO123 address 10.10.13.1 |crypto isakmp key CISCO123 address 10.10.13.1     |crypto isakmp key CISCO123 address 10.10.12.1   |
|!    |!     |!  |
|crypto ipsec transform-set GRE esp-3des esp-md5-hmac    |crypto ipsec transform-set GRE esp-3des esp-md5-hmac     |crypto ipsec transform-set GRE esp-3des esp-md5-hmac  |
|mode tunnel    |mode tunnel     |mode tunnel  |
|crypto ipsec profile GRE-IPSEC    |crypto ipsec profile GRE-IPSEC     |crypto ipsec profile GRE-IPSEC  |
|set transform-set GRE    |set transform-set GRE     |set transform-set GRE  |
|!    |!     |!  |
|interface tunnel 1    |interface tunnel 1     |interface tunnel 2  |
|tunnel protection ipsec profile GRE-IPSEC    |tunnel protection ipsec profile GRE-IPSEC     |tunnel protection ipsec profile GRE-IPSEC  |
|interface tunnel 2    |interface tunnel 3    |interface tunnel 3 |
|tunnel protection ipsec profile GRE-IPSEC    |tunnel protection ipsec profile GRE-IPSEC     |tunnel protection ipsec profile GRE-IPSEC  |
|exit    |exit     |exit  |


# Настройка NAT R1, R2, R3
| R1              | R2             |R3             | 
| -------------         | -------------         | -------------         |
|interface Gi0/1    |interface Gi0/1     |interface Gi0/1  |
|ip nat inside    |ip nat inside     |ip nat inside  |
|!    |!     |!  |
|interface Gi0/0    |interface Gi0/0     |interface Gi0/0  |
|ip nat outside    |ip nat outside     |ip nat outside  |
|!    |!     |!  |
|access-list 1 permit 192.168.11.0 0.0.0.255    |access-list 1 permit 192.168.12.0 0.0.0.255     |access-list 1 permit 192.168.13.0 0.0.0.255  |
|ip nat inside source list 1 interface Gi0/0 overload    |ip nat inside source list 1 interface Gi0/0 overload     |ip nat inside source list 1 interface Gi0/0 overload |
|!    |!     |!  |
|ip nat inside source static tcp 192.168.11.2 80 10.10.11.1 80    |ip nat inside source static tcp 192.168.12.2 80 10.10.12.1 80     |
ip nat inside source static tcp 192.168.13.2 80 10.10.13.1 80  |


