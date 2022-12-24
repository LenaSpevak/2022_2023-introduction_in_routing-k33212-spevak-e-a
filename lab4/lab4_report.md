#### University: [ITMO University](https://##3itmo.ru/ru/)
 #### Faculty: [FICT](https://fict.itmo.ru)
 #### Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)
 #### Year: 2022/2023
 ####  Group: K33212
 #### Author: Spevak Elena Aleksandrovna
 #### Lab: Lab4
 #### Date of create: 13.12.2022
 #### Date of finished: 2.12.2022

# **Отчёт по лабораторной работе №4**

# Эмуляция распределенной корпоративной сети связи, настройка iBGP, организация L3VPN, VPLS

# Цель работы
Изучить протоколы BGP, MPLS и правила организации L3VPN и VPLS.

# Ход работы
 В Containerlab была создана IP/MPLS сеть для компании "RogaIKopita Games", топология которой описана в [yaml файле](https://github.com/LenaSpevak/2022_2023-introduction_in_routing-k33212-spevak-e-a/blob/main/lab4/task4.yaml). 

 Схема сети:
 ![schema](https://github.com/LenaSpevak/2022_2023-introduction_in_routing-k33212-spevak-e-a/blob/main/lab4/screenshots/lab4.drawio.png)

На устройствах были настроены IP адреса на интерйесах, OSPF и MPLS, iBGP с route  reflect кластером.
 
Для начала был настроен L3VPN. 
На роутерах были созданы Loopback интерфейсы, на которых настроены адреса. На трех роутерах в центре схемы настроены iBGP RR Cluster, VRF причём на интерфейсах между ними route-reflect=yes, а на итерфейсах, которые связывают роутеры с подключенными комьпютерами, route-reflect=no. При настройке Route Distinguisher и Route Target 

Тексты конфигураций каждого из сетевых устройств:
- Роутер R01.NY:
```/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default redistribute-connected=yes router-id=1.1.1.1
/routing ospf instance
set [ find default=yes ] router-id=1.1.1.1
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=1.1.1.1 interface=Lo network=1.1.1.1
add address=172.10.1.1/30 interface=ether3 network=172.10.1.0
add address=192.168.20.1/30 interface=ether2 network=192.168.20.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether2 \
    route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether3
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=2.2.2.2 remote-as=65530 \
    update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```
- Роутер R01.LND:
```/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=2.2.2.2
/routing ospf instance
set [ find default=yes ] router-id=2.2.2.2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=2.2.2.2 interface=Lo network=2.2.2.2
add address=172.10.1.2/30 interface=ether2 network=172.10.1.0
add address=172.10.2.1/30 interface=ether3 network=172.10.2.0
add address=172.10.3.1/30 interface=ether4 network=172.10.3.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=5.5.5.5 remote-as=65530 \
    route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=3.3.3.3 remote-as=65530 \
    route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=1.1.1.1 remote-as=65530 \
    update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.LND
```
- Роутер R01.HKI:
```/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=3.3.3.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=172.10.2.2/30 interface=ether2 network=172.10.2.0
add address=3.3.3.3 interface=Lo network=3.3.3.3
add address=172.10.5.1/30 interface=ether3 network=172.10.5.0
add address=172.10.4.1/30 interface=ether4 network=172.10.4.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=2.2.2.2 remote-as=65530 \
    route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=5.5.5.5 remote-as=65530 \
    route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=4.4.4.4 remote-as=65530 \
    update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.HKI
```
- Роутер R01.SPB:
```/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=4.4.4.4
/routing ospf instance
set [ find default=yes ] router-id=4.4.4.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=4.4.4.4 interface=Lo network=4.4.4.4
add address=172.10.5.2/30 interface=ether2 network=172.10.5.0
add address=192.168.10.1/30 interface=ether3 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether3 \
    route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether2
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=3.3.3.3 remote-as=65530 \
    update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB
```
- Роутер R01.LBN:
```/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=5.5.5.5
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=5.5.5.5 interface=Lo network=5.5.5.5
add address=172.10.3.2/30 interface=ether2 network=172.10.3.0
add address=172.10.6.1/30 interface=ether4 network=172.10.6.0
add address=172.10.4.2/30 interface=ether3 network=172.10.4.0
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=2.2.2.2 remote-as=65530 \
    route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=3.3.3.3 remote-as=65530 \
    route-reflect=yes update-source=Lo
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=6.6.6.6 remote-as=65530 \
    update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.LBN
```
- Роутер R01.SVL:
```/interface bridge
add name=Lo
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default redistribute-connected=yes
/routing ospf instance
set [ find default=yes ] router-id=6.6.6.6
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=6.6.6.6 interface=Lo network=6.6.6.6
add address=172.10.6.2/30 interface=ether2 network=172.10.6.0
add address=192.168.30.1/30 interface=ether3 network=192.168.30.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether3 \
    route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes
/mpls ldp interface
add interface=ether2
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=5.5.5.5 remote-as=65530 \
    update-source=Lo
/routing ospf network
add area=backbone
/system identity
set name=R01.SVL
```
Проверка связанности между VRF:

![Ping NY](https://github.com/LenaSpevak/2022_2023-introduction_in_routing-k33212-spevak-e-a/blob/main/lab4/screenshots/ping_NY.png)

![Ping SPB](https://github.com/LenaSpevak/2022_2023-introduction_in_routing-k33212-spevak-e-a/blob/main/lab4/screenshots/ping_SPB.png)

![Pinf SVL](https://github.com/LenaSpevak/2022_2023-introduction_in_routing-k33212-spevak-e-a/blob/main/lab4/screenshots/ping_SVL.png)


# Вывод
Были изучены протоколы BGP, MPLS и правила организации L3VPN и VPLS. Полученные знания были применены при проектировании  IP/MPLS сеть связи для компании "RogaIKopita Games".
