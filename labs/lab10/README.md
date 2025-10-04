## iBGP

### Цели:
1) Настроите iBGP в офисом Москва между маршрутизаторами R14 и R15; 
2) Настроите iBGP в провайдере Триада, с использованием RR;
3) Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас;
4) Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно;
5) Все сети в лабораторной работе должны иметь IP связность.

Настройка соседства BGP на маршрутизаторе R14
```
R14(config-router)#neighbor 2001:2025:1111:8::1 remote-as 1001
R14(config-router)#address-family ipv6
R14(config-router-af)#neighbor 2001:2025:1111:8::1 ac
R14(config-router-af)#neighbor 2001:2025:1111:8::1 activate
```

Настройка соседства BGP на маршрутизаторе R15
```
R15(config-router)#neighbor 2001:2025:1111:8:: remote-as 1001
R15(config-router)#address-family ipv6
R15(config-router-af)#neighbor 2001:2025:1111:8:: activate
*Oct  2 20:05:19.424: %BGP-5-ADJCHANGE: neighbor 2001:2025:1111:8:: Up
```

Далее, пропишем статические маршруты в сторону провайдеров,отдадим с R15 в сторону R14 маршрут по-умолчанию и на R14 пропишем статический маршрут в сторону провайдера с меньшей метрикой.

```
R14(config)#ipv6 route ::/0 2001:2025:ABCD:FF00::1 250
R14(config)#do sh ipv6 route
IPv6 Routing Table - default - 17 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       RL - RPL, O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1
       OE2 - OSPF ext 2, ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       la - LISP alt, lr - LISP site-registrations, ld - LISP dyn-eid
       lA - LISP away, a - Application
B   ::/0 [200/0]
     via 2001:2025:ABCD:FF03::1
C   2001:2025:1111:2::/127 [0/0]
     via Ethernet0/0, directly connected
L   2001:2025:1111:2::/128 [0/0]
     via Ethernet0/0, receive
OI  2001:2025:1111:3::/127 [110/20]
     via FE80:1111::4, Ethernet1/0
OI  2001:2025:1111:4::/127 [110/20]
     via FE80:1111::4, Ethernet1/0
C   2001:2025:1111:5::/127 [0/0]
     via Ethernet0/1, directly connected
L   2001:2025:1111:5::/128 [0/0]
```

```
R15(config)#router bgp 1001
R15(config-router)# address-family ipv6
R15(config)#ipv6 route ::/0 2001:2025:ABCD:FF03::1
R15(config-router-af)# network ::/0
```

Далее, в Триаде я настроил анаонс Loopback-адресов у каждого роутера. Построим IBGP соседство и сделаем R23 Route-reflector'ом

Настройка BGP на маршрутизаторе R23
```
R23(config-router)#neighbor 2001:DEAD:BEEF::13 remote-as 520
R23(config-router)#neighbor 2001:DEAD:BEEF::13 update-source loopback 1
R23(config-router)#address-family ipv6
R23(config-router-af)#neighbor 2001:DEAD:BEEF::13 activate
R23(config-router-af)#neighbor 2001:DEAD:BEEF::13 route-reflector-client

R23(config-router)#neighbor 2001:DEAD:BEEF::14 remote-as 520
R23(config-router)#neighbor 2001:DEAD:BEEF::14 update-source loopback 1
R23(config-router)#address-family ipv6
R23(config-router-af)#neighbor 2001:DEAD:BEEF::14 activate
R23(config-router-af)#neighbor 2001:DEAD:BEEF::14 route-reflector-client

R23(config-router)#neighbor 2001:DEAD:BEEF::15 remote-as 520
R23(config-router)#neighbor 2001:DEAD:BEEF::15 update-source loopback 1
R23(config-router)#address-family ipv6
R23(config-router-af)#neighbor 2001:DEAD:BEEF::15 activate
R23(config-router-af)#neighbor 2001:DEAD:BEEF::15 route-reflector-client

R23(config-router-af)#do sh bgp ipv6 uni sum
BGP router identifier 23.23.23.23, local AS number 520
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:2025:ABCD:FF02::
                4          101       4       4        1    0    0 00:00:33        0
2001:DEAD:BEEF::13
                4          520      29      30        1    0    0 00:23:34        0
2001:DEAD:BEEF::14
                4          520      18      16        1    0    0 00:13:05        0
2001:DEAD:BEEF::15
                4          520      10       8        1    0    0 00:06:08        0
```

Настройка BGP на маршрутизаторе R24
```
R24(config-router)#neighbor 2001:DEAD:BEEF::12 remote-as 520
R24(config-router)#neighbor 2001:DEAD:BEEF::12 update-source loopback 1
R24(config-router)#address-family ipv6
R24(config-router-af)#neighbor 2001:DEAD:BEEF::12 activate
R24(config-router-af)#
*Oct  4 21:46:37.572: %BGP-5-ADJCHANGE: neighbor 2001:DEAD:BEEF::12 Up
```

Настройка BGP на маршрутизаторе R25
```
R25(config)#router bgp 520
R25(config-router)#bgp router-id 25.25.25.25
R25(config-router)#neighbor 2001:DEAD:BEEF::12 remote-as 520
R25(config-router)#neighbor 2001:DEAD:BEEF::12 update-source loopback 1
R25(config-router)#address-family ipv6
R25(config-router-af)#neighbor 2001:DEAD:BEEF::12 activate
R25(config-router-af)#
*Oct  4 21:56:59.651: %BGP-5-ADJCHANGE: neighbor 2001:DEAD:BEEF::12 Up
```

Настройка BGP на маршрутизаторе R26
```
R26(config)#router bgp 520
R26(config-router)# neighbor 2001:DEAD:BEEF::12 remote-as 520
R26(config-router)# neighbor 2001:DEAD:BEEF::12 update-source Loopback1
R26(config-router)# address-family ipv6
R26(config-router-af)#neighbor 2001:DEAD:BEEF::12 activate
*Oct  4 22:04:03.560: %BGP-5-ADJCHANGE: neighbor 2001:DEAD:BEEF::12 Up
```

В рамках настройки Санкт-Петербурга так как не стоит задача отказоустойчивости, я не вижу смысла использовать BGP и просто двумя статическими маршрутами через 2 провайдера активировать ECMP.

Настройка на маршрутизаторе R18
```
R18(config)# ipv6 route ::/0 2001:2025:ABCD:FF05::1
R18(config)#
R18(config)# ipv6 route ::/0 2001:2025:ABCD:FF09::1
R18(config)#
R18(config)#do sh ipv6 route
IPv6 Routing Table - default - 13 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       RL - RPL, O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1
       OE2 - OSPF ext 2, ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       la - LISP alt, lr - LISP site-registrations, ld - LISP dyn-eid
       lA - LISP away, a - Application
S   ::/0 [1/0]
     via 2001:2025:ABCD:FF05::1
     via 2001:2025:ABCD:FF09::1
```
