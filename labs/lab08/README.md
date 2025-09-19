## EIGRP

### Цели:
1) В офисе Санкт-Петербург настроить EIGRP;
2) R32 получает только маршрут по умолчанию;
3) R16-17 анонсируют только суммарные префиксы;
4) Использовать EIGRP named-mode для настройки сети.

### Топология Санкт-Петербурга
![Картинка](./pictures/lab08-SPB-topology.png)

Итоговая конфигурация устройств.

Настройка EIGRP на маршрутизаторе R16
```
R16(config)#router eigrp OTUS_NETWORK
R16(config-router)#address-family ipv6 unicast autonomous-system 100
R16(config-router-af)#eigrp router-id 6.6.6.6
R16(config-router-af)#af-interface Ethernet0/1
R16(config-router-af-interface)#summary-address 2001:2025:3333::/64
R16(config-router-af)#af-interface Ethernet0/2
R16(config-router-af-interface)#summary-address 2001:2025:3333::/64
R16(config-router-af-interface)#af-interface Ethernet0/0               
R16(config-router-af-interface)#summary-address 2001:2025:3333::/64
```

Настройка EIGRP на маршрутизаторе R17
```
R17(config)#ipv6 route ::/0 Null0
R17(config)#router eigrp OTUS_NETWORK
R17(config-router)#address-family ipv6 unicast autonomous-system 100
R17(config-router-af)#topology base 
R17(config-router-af-topology)#redistribute static 
R17(config-router-af)#eigrp router-id 7.7.7.7
R17(config-router-af)#af-interface Ethernet0/1
R17(config-router-af-interface)#summary-address 2001:2025:3333::/64
R17(config-router-af-interface)#af-interface Ethernet0/2           
R17(config-router-af-interface)#summary-address 2001:2025:3333::/64
R17(config-router-af-interface)#af-interface Ethernet0/0           
R17(config-router-af-interface)#summary-address 2001:2025:3333::/64
```

Настройка EIGRP на маршрутизаторе R18
```
R18(config)#router eigrp OTUS_NETWORK
R18(config-router)# address-family ipv6 unicast autonomous-system 100
R18(config-router-af)#topology base
R18(config-router-af-topology)#eigrp router-id 8.8.8.8
R18(config-router-af)#
*Sep 19 15:08:34.046: %DUAL-5-NBRCHANGE: EIGRP-IPv6 100: Neighbor FE80:3333::5 (Ethernet0/1) is up: new adjacency
*Sep 19 15:08:34.054: %DUAL-5-NBRCHANGE: EIGRP-IPv6 100: Neighbor FE80:3333::4 (Ethernet0/0) is up: new adjacency
```

Настройка EIGRP на маршрутизаторе R32
```
R32(config)#ipv6 prefix-list DEFAULT_ONLY permit ::/0
R32(config)#ipv6 prefix-list DEFAULT_ONLY deny ::/0 le 128
R32(config)#router eigrp OTUS_NETWORK
R32(config-router)#address-family ipv6 unicast autonomous-system 100
R32(config-router-af)#eigrp router-id 3.2.3.2
R32(config-router-af)# topology base
R32(config-router-af-topology)#distribute-list prefix-list DEFAULT_ONLY in
R32(config-if)#do show ipv6 route eigrp
IPv6 Routing Table - default - 5 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       RL - RPL, O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1
       OE2 - OSPF ext 2, ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       la - LISP alt, lr - LISP site-registrations, ld - LISP dyn-eid
       lA - LISP away, a - Application
EX  ::/0 [170/1024512]
     via FE80:3333::5, Ethernet0/0
```
