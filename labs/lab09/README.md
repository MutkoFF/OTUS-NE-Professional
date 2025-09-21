## BGP. Продолжение

### Цели:
1) Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
2) Настроите eBGP между провайдерами Киторн и Ламас.
3) Настроите eBGP между Ламас и Триада.
4) Настроите eBGP между офисом С.-Петербург и провайдером Триада.
5) Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.

Настройка BGP на маршрутизаторе R14
```
R14(config)#router bgp 1001
R14(config-router)#bgp router-id 14.14.14.14
R14(config-router)#neighbor 2001:2025:ABCD:FF00::1 remote-as 101
R14(config-router)#address-family ipv6 unicast
R14(config-router-af)#neighbor 2001:2025:ABCD:FF00::1 activate 
R14(config-router-af)#do sh bgp ipv6 unicast summary
BGP router identifier 14.14.14.14, local AS number 1001
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:2025:ABCD:FF00::1
                4          101       6       6        1    0    0 00:02:40        0
```

Настройка BGP на маршрутизаторе R15
```
R15(config)#router bgp 1001
R15(config-router)#bgp router-id 15.15.15.15
R15(config-router)#neighbor 2001:2025:ABCD:FF03::1 remote-as 301
R15(config-router)#address-family ipv6 unicast
R15(config-router-af)#neighbor 2001:2025:ABCD:FF03::1 activate 
R15(config-router-af)#do sh bgp ipv6 unicast summary
BGP router identifier 15.15.15.15, local AS number 1001
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:2025:ABCD:FF03::1
                4          301       4       4        1    0    0 00:00:52        0
```

Настройка BGP на маршрутизаторе R21
```
R21(config)#router bgp 301
R21(config-router)#bgp router-id 21.21.21.21
R21(config-router)#neighbor 2001:2025:ABCD:FF03:: remote-as 1001
R21(config-router)#address-family ipv6 unicast
R21(config-router-af)#neighbor 2001:2025:ABCD:FF03:: activate 
R21(config-router)#neighbor 2001:2025:ABCD:FF04::1 remote-as 520
R21(config-router)#address-family ipv6 unicast                  
R21(config-router-af)#neighbor 2001:2025:ABCD:FF04::1 activate 
R21(config-router)#neighbor 2001:2025:ABCD:FF01:: remote-as 101
R21(config-router)#address-family ipv6
R21(config-router-af)#neighbor 2001:2025:ABCD:FF01:: activate
```

Настройка BGP на маршрутизаторе R22
```
R22(config)#router bgp 101
R22(config-router)#bgp router-id 22.22.22.22
R22(config-router)#neighbor 2001:2025:ABCD:FF00:: remote-as 1001
R22(config-router)#address-family ipv6 unicast
R22(config-router-af)#neighbor 2001:2025:ABCD:FF00:: activate 
R22(config-router)#neighbor 2001:2025:abcd:ff04::1 remote-as 520
R22(config-router)#address-family ipv6 
R22(config-router-af)#neighbor 2001:2025:abcd:ff04::1 activate 
R22(config-router)# neighbor 2001:2025:abcd:ff01::1 remote-as 301
R22(config-router)# address-family ipv6
R22(config-router-af)#neighbor 2001:2025:abcd:ff01::1 activate
```
Настройка BGP на маршрутизаторе R23
```
R23(config)#router bgp 520
R23(config-router)#bgp router-id 23.23.23.23

```

Настройка BGP на маршрутизаторе R24
```
R24(config)#router bgp 520
R24(config-router)#bgp router-id 24.24.24.24
R24(config-router)#neighbor 2001:2025:ABCD:FF05:: remote-as 2042
R24(config-router)#address-family ipv6 unicast
R24(config-router-af)#neighbor 2001:2025:ABCD:FF05:: activate
R24(config-router)#neighbor 2001:2025:ABCD:FF04:: remote-as 301
R24(config-router)# address-family ipv6
R24(config-router-af)#neighbor 2001:2025:ABCD:FF04:: activate

```

Настройка BGP на маршрутизаторе R26
```
R26(config)#router bgp 520
R26(config-router)#bgp router-id 26.26.26.26
R26(config-router)#neighbor 2001:2025:ABCD:FF09:: remote-as 2042
R26(config-router)#address-family ipv6 unicast
R26(config-router-af)#neighbor 2001:2025:ABCD:FF09:: activate
```

Настройка BGP на маршрутизаторе R18
```
R18(config)#router bgp 2042
R18(config-router)#bgp router-id 18.18.18.18
R18(config-router)#neighbor 2001:2025:ABCD:FF05::1 remote-as 520
R18(config-router)#address-family ipv6 unicast
R18(config-router-af)#neighbor 2001:2025:ABCD:FF05::1 activate 
R18(config-router)#neighbor 2001:2025:ABCD:FF09::1 remote-as 520
R18(config-router-af)#neighbor 2001:2025:ABCD:FF09::1 activate 
```

После настройки BGP пирингов между роутерами, настроим анонс loopback-адресов так, чтобы R14 и R15 иогли добраться до R18 и наоборот. 

Настройка анонса loopback на R14
```
R14(config)#router bgp 1001
R14(config-router)#address-family ipv6
R14(config-router-af)#network 2001:DEAD:BEEF::3/128
```

Настройка анонса loopback на R15
```
R15(config)#router bgp 1001
R15(config-router)# address-family ipv6
R15(config-router-af)#network 2001:dead:beef::4/128
```

Настройка анонса loopback на R18
```
R18(config)#router bgp 2042
R18(config-router)# address-family ipv6
R18(config-router-af)#network 2001:DEAD:BEEF::10/128
```

Далее, проверим связанность между роутерами Москвы и Санкт-Петербурга
R14
```
R14#ping 2001:dead:beef::10 source 2001:DEAD:BEEF::3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DEAD:BEEF::10, timeout is 2 seconds:
Packet sent with a source address of 2001:DEAD:BEEF::3
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

R15
```
R15#ping 2001:dead:beef::10 source 2001:DEAD:BEEF::4
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DEAD:BEEF::10, timeout is 2 seconds:
Packet sent with a source address of 2001:DEAD:BEEF::4
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

R18
```
R18#ping ipv6  2001:DEAD:BEEF::3 source  2001:dead:beef::10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DEAD:BEEF::3, timeout is 2 seconds:
Packet sent with a source address of 2001:DEAD:BEEF::10
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R18#ping ipv6  2001:DEAD:BEEF::4 source  2001:dead:beef::10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DEAD:BEEF::4, timeout is 2 seconds:
Packet sent with a source address of 2001:DEAD:BEEF::10
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```