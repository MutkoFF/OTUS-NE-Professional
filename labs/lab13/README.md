## VPN. GRE. DmVPN

### Цели:
1) Настроите GRE между офисами Москва и С.-Петербург;
2) Настроите DMVMN между Москва и Чокурдах, Лабытнанги;
3) Все узлы в офисах в лабораторной работе должны иметь IP связность;
3) План работы и изменения зафиксированы в документации.

Для начала настроим GRE туннель между R15 в Москве и R18 в Санкт-Петербурге

```
R15(config)#interface Tunnel0
*Nov 21 20:15:15.987: %LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to down
R15(config-if)#ipv6 add 2001:2025:5555::1/127
R15(config-if)# tunnel source Ethernet0/2
R15(config-if)# tunnel destination 2001:2025:ABCD:FF05::
R15(config-if)#tunnel mode gre ipv6
*Nov 21 20:26:43.116: %LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to up
```
```
R18(config)#interface Tunnel0
R18(config-if)#ipv6 add 2001:2025:5555::/127
R18(config-if)#tunnel source Ethernet0/2
R18(config-if)#tunnel destination 2001:2025:ABCD:FF03::
R18(config-if)# tunnel mode gre ipv6
*Nov 21 20:26:37.788: %LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel0, changed state to up
R18(config-if)#do sh int des
Interface                      Status         Protocol Description
Et0/0                          up             up
Et0/1                          up             up
Et0/2                          up             up
Et0/3                          up             up
Lo1                            up             up
Tu0                            up             up
```
После поднятия туннелей настроим BGP внутри GRE между R15 и R18 и также пропишем статитечские ссумаризированный маршруты (сначала хотел поднять BGP, но так как в задании нет отказоустойчивости, статиками быстрее будет)
```
R15(config-if)#router bgp 1001
R15(config-router)#neighbor 2001:2025:5555:: remote-as 2042
R15(config-router)# address-family ipv6
R15(config-router-af)#neighbor 2001:2025:5555:: activate
*Nov 21 20:34:03.778: %BGP-5-ADJCHANGE: neighbor 2001:2025:5555:: Up
R15(config)# ipv6 route 2001:2025:3333::/48 2001:2025:5555::
```

```
R18(config-if)#router bgp 2042
R18(config-router)#neighbor 2001:2025:5555::1 remote-as 1001
R18(config-router)# address-family ipv6
R18(config-router-af)#neighbor 2001:2025:5555::1 activate
*Nov 21 20:34:03.779: %BGP-5-ADJCHANGE: neighbor 2001:2025:5555::1 Up
R18(config)# ipv6 route 2001:2025:1111::/48 2001:2025:5555::1
```

Далее, приступим к настройке DMVPN между Москва и Чокурдах, Лабытнанги. R14 в Москве будет выступать в качестве HUB-маршрутизаторa, а R27 и R28 будут Spoke-роутерами. И для проверки добавим маршруты до loopback-интерфейса роутера в Лабытнанги и локальной подсети в Чокурдах.
```
R14(config)#interface Tunnel1
*Nov 22 19:36:58.744: %LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel1, changed state to down
R14(config-if)#ipv6 add 2001:2025:6666::1/48
R14(config-if)#no ip redirects
R14(config-if)#ipv6 nhrp authentication nhrp1234
R14(config-if)#ipv6 nhrp network-id 1
R14(config-if)# load-interval 30
R14(config-if)# keepalive 5 10
R14(config-if)#tunnel source Ethernet0/2
R14(config-if)# tunnel mode gre multipoint
R14(config-if)#ipv6 nhrp map multicast dynamic
R14(config-if)#ipv6 mtu 1440
R14(config-if)#exit
R14(config)#ipv6 route 2001:2025:2222:50::/64 2001:2025:6666::2
R14(config)#ipv6 route 2001:dead:beef::16/128 2001:2025:6666::3
```

```
R28(config)#interface Tunnel1
R28(config-if)#ipv6 add 2001:2025:6666::2/48
R28(config-if)#no ip redirects
R28(config-if)#ipv6 nhrp map multicast dynamic
R28(config-if)#tunnel source Et0/1
R28(config-if)#tunnel mode gre multipoint
R28(config-if)#ipv6 nhrp authentication nhrp1234
R28(config-if)#ipv6 nhrp map 2001:2025:6666::1/128 2001:2025:ABCD:FF00::
R28(config-if)#ipv6 nhrp network-id 1
R28(config-if)#ipv6 nhrp nhs 2001:2025:6666::1/128
R28(config-if)#ipv6 nhrp nhs 2001:2025:6666::1
R28(config-if)#ipv6 nhrp registration no-unique
R28(config-if)#ipv6 nhrp map multicast 2001:2025:ABCD:FF00::
R28(config-if)# ip mtu 1440
R28(config-if)# load-interval 30
R28(config-if)# keepalive 5 10
R28(config-if)#exit
R28(config-if)#ipv6 route 2001:2025:1111::/48 2001:2025:6666::1
```

И дальше я несколько часов потратил на дебаг, потому что у меня не устанавливалось динамическое DMVPN соседство. Я пробовал создавать статическую NHRP запись, но spoke не может зарегистироваться:
```
R28(config)#do show ipv6 nhrp
2001:2025:6666::1/128 via 2001:2025:6666::1
   Tunnel1 created 00:05:46, never expire
   Type: static, Flags:
   NBMA address: 2001:2025:ABCD:FF00::
R28(config)#
R28(config)#
R28(config)#
R28(config)#do show dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        T1 - Route Installed, T2 - Nexthop-override
        C - CTS Capable, I2 - Temporary
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel1, IPv6 NHRP Details
Type:Spoke, Total NBMA Peers (v4/v6): 1
    1.Peer NBMA Address: 2001:2025:ABCD:FF00::
        Tunnel IPv6 Address: 2001:2025:6666::1
        IPv6 Target Network: 2001:2025:6666::1/128
        # Ent: 1, Status: INTF, UpDn Time: 00:05:59, Cache Attrib: S
```

И после изучения других кейсов я пришёл к выводу, что в Cisco IOL (IOS-on-Linux) 15.7(3)M2 (который я использую) не поддерживает корректную DAD на mGRE IPv6 интерфейсах. Также пишут, что на реальном железе это уже исправили, но IOL — это урезанная лабораторная сборка.