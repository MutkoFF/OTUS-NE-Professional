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

```