## IS-IS

### Цели:
1) Настроите IS-IS в ISP Триада;
2) R23 и R25 находятся в зоне 2222;
3) R24 находится в зоне 24;
4) R26 находится в зоне 26.

### Топология Триады
![Картинка](./pictures/lab07-Triada-topology.jpg)

**Шаг 1. Настроить маршрутизаторы R14-R15 находятся в зоне 0 - backbone**
```
R14(config)#router ospfv3 1
R14(config-router)#router-id 1.1.1.1
R14(config-router)#interface Et1/0
R14(config-if)#ipv6 ospf 1 area 0
R14(config-if)#
*Jul 30 19:50:16.136: %OSPFv3-5-ADJCHG: Process 1, IPv6, Nbr 2.2.2.2 on Ethernet1/0 from LOADING to FULL, Loading Done
R14(config-if)#end
R14#sh ipv6 ospf neighbor 

            OSPFv3 Router with ID (1.1.1.1) (Process ID 1)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
2.2.2.2           1   FULL/DR         00:00:39    7               Ethernet1/0

```

```
R15(config)#router ospfv3 1
R15(config-router)#router-id 2.2.2.2
R15(config-router)#interface Et1/0
R15(config-if)#ipv6 ospf 1 area 0
R15(config-if)#end
R15(config)# 
*Jul 30 19:50:16.136: %OSPFv3-5-ADJCHG: Process 1, IPv6, Nbr 1.1.1.1 on Ethernet1/0 from LOADING to FULL, Loading Done
R15(config)#do sh ipv6 ospf neighbor

            OSPFv3 Router with ID (2.2.2.2) (Process ID 1)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
1.1.1.1           1   FULL/BDR        00:00:34    7               Ethernet1/0

```