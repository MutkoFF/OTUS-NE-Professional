## OSPF

### Цели:
1) Настроить OSPF офисе Москва;
2) Разделить сеть на зоны;
3) Настроить фильтрацию между зонами.

### Топология Москвы
![Картинка](./pictures/lab06-Moscow-topology.jpg)

**Шаг 1. Настроить маршрутизаторы R14-R15 находятся в зоне 0 - backbone**
```
R14(config)#router ospfv3 1
R14(config-router)#router-id 1.1.1.1
R14(config-router)#interface Ethernet0/0
R14(config-if)#ipv6 ospf 1 area 0
R14(config-if)#interface Ethernet0/1
R14(config-if)#ipv6 ospf 1 area 0   
R14(config-if)#interface Loopback1
R14(config-if)#ipv6 ospf 1 area 0
```

```
R15(config)#router ospfv3 1
R15(config-router)#router-id 3.3.3.3
R15(config-router)#interface Loopback0
R15(config-if)# ipv6 ospf 1 area 0
R15(config-if)#interface Ethernet0/0
R15(config-if)# ipv6 ospf 1 area 0  
R15(config-if)#interface Ethernet0/1
R15(config-if)# ipv6 ospf 1 area 0 
```