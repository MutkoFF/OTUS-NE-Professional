## Основные протоколы сети интернет

### Цели:
1) Настроить DHCP в офисе Москва;
2) Настроить синхронизацию времени в офисе Москва;
3) Настроить NAT в офисе Москва, C.-Перетбруг и Чокурдах.

Так как я использую IPv6, вместо DHCP у меня будет использоваться SLAAC. 

Настроим на R12 и R13 и DHCP для получения конфигурации по SLAAC на VPC1 в VLAN10.
```
R12(config)#ipv6 dhcp pool OTUS-vlan10
R12(config-dhcpv6)#dns-server 2001:2025:1111:0010::ffff
R12(config-dhcpv6)#domain-name otus.ru
R12(config-dhcpv6)#exit
R12(config)#interface Ethernet0/0
R12(config-if)#no sh
R12(config-if)#interface Ethernet0/0.10
R12(config-subif)#encapsulation dot1Q 10
R12(config-subif)#ipv6 nd other-config-flag
R12(config-subif)#ipv6 dhcp server OTUS-vlan10
```

```
R13(config)#ipv6 dhcp pool OTUS-vlan10
R13(config-dhcpv6)#dns-server 2001:2025:1111:0010::ffff
R13(config-dhcpv6)#domain-name otus.ru
R13(config-dhcpv6)#exit
R13(config)#
R13(config)#interface Ethernet0/1
R13(config-if)#no sh
R13(config-if)#exit
R13(config)#interface Ethernet0/1.10
R13(config-subif)#ipv6 nd other-config-flag
R13(config-subif)#ipv6 dhcp server OTUS-vlan10
R13(config-subif)# encapsulation dot1Q 10
```

Далее, зайдём на VPC1 в 10 VLAN, получим конфигурацию по SLAAC и пинганём HSRP-адрес.
```
VPC1> ip auto
GLOBAL SCOPE      : 2001:2025:1111:10:2050:79ff:fe66:68e3/64
ROUTER LINK-LAYER : 00:05:73:a0:00:01

VPC1> show ipv6

NAME              : VPC1[1]
LINK-LOCAL SCOPE  : fe80::250:79ff:fe66:68e3/64
GLOBAL SCOPE      : 2001:2025:1111:10:2050:79ff:fe66:68e3/64
DNS               : 
ROUTER LINK-LAYER : 00:05:73:a0:00:01
MAC               : 00:50:79:66:68:e3
LPORT             : 20000
RHOST:PORT        : 127.0.0.1:30000
MTU:              : 1500

VPC1> ping 2001:2025:1111:10::FFFF

2001:2025:1111:10::FFFF icmp6_seq=1 ttl=64 time=9.665 ms
2001:2025:1111:10::FFFF icmp6_seq=2 ttl=64 time=0.754 ms
2001:2025:1111:10::FFFF icmp6_seq=3 ttl=64 time=0.746 ms
2001:2025:1111:10::FFFF icmp6_seq=4 ttl=64 time=0.818 ms
2001:2025:1111:10::FFFF icmp6_seq=5 ttl=64 time=0.855 ms

VPC1> 
```

Затем, настроим на R12 и R13 и DHCP для получения конфигурации по SLAAC на VPC7 в VLAN20.
```
R13(config)#ipv6 dhcp pool OTUS-vlan20
R13(config-dhcpv6)#dns-server 2001:2025:1111:0020::ffff
R13(config-dhcpv6)#domain-name otus.ru
R13(config)#interface Ethernet0/0.20
R13(config-subif)# standby version 2
R13(config-subif)# standby 2 ipv6 FE80:1111::FFFF
R13(config-subif)# standby 2 ipv6 2001:2025:1111:20::FFFF/64
R13(config-subif)# standby 2 priority 150
R13(config-subif)# standby 2 preempt
R13(config-subif)# ipv6 address FE80:1111::2 link-local
R13(config-subif)# ipv6 address 2001:2025:1111:20::2/64
R13(config-subif)# ipv6 nd other-config-flag
R13(config-subif)# ipv6 dhcp server OTUS-vlan20
*Jul 20 11:54:17.069: %HSRP-5-STATECHANGE: Ethernet0/0.20 Grp 2 state Standby -> Active
```
```
R12(config)#ipv6 dhcp pool OTUS-vlan20
R12(config-dhcpv6)#dns-server 2001:2025:1111:0020::ffff
R12(config-dhcpv6)#domain-name otus.ru
R12(config)#interface Ethernet0/1.20
R12(config-subif)# encapsulation dot1Q 20
R12(config-subif)# standby version 2
R12(config-subif)# ipv6 address FE80:1111::1 link-local
R12(config-subif)# ipv6 address 2001:2025:1111:20::1/64
R12(config-subif)# ipv6 nd other-config-flag
R12(config-subif)# ipv6 dhcp server OTUS-vlan20
R12(config-subif)# standby 2 ipv6 FE80:1111::FFFF
R12(config-subif)# standby 2 ipv6 2001:2025:1111:20::FFFF/64
R12(config-subif)#
R12(config-subif)#
*Jul 20 11:55:57.143: %HSRP-5-STATECHANGE: Ethernet0/1.20 Grp 2 state Speak -> Standby
```

Далее, зайдём на VPC7 в 20 VLAN, получим конфигурацию по SLAAC и пинганём HSRP-адрес
```
VPC7> ip auto  
GLOBAL SCOPE      : 2001:2025:1111:20:2050:79ff:fe66:68e4/64
ROUTER LINK-LAYER : 00:05:73:a0:00:02

VPC7> show ipv6

NAME              : VPC7[1]
LINK-LOCAL SCOPE  : fe80::250:79ff:fe66:68e4/64
GLOBAL SCOPE      : 2001:2025:1111:20:2050:79ff:fe66:68e4/64
DNS               : 
ROUTER LINK-LAYER : 00:05:73:a0:00:02
MAC               : 00:50:79:66:68:e4
LPORT             : 20000
RHOST:PORT        : 127.0.0.1:30000
MTU:              : 1500

VPC7> ping 2001:2025:1111:20::FFFF

2001:2025:1111:20::FFFF icmp6_seq=1 ttl=64 time=0.593 ms
2001:2025:1111:20::FFFF icmp6_seq=2 ttl=64 time=0.893 ms
2001:2025:1111:20::FFFF icmp6_seq=3 ttl=64 time=1.023 ms
2001:2025:1111:20::FFFF icmp6_seq=4 ttl=64 time=1.172 ms
2001:2025:1111:20::FFFF icmp6_seq=5 ttl=64 time=1.197 ms
```

После этого, настроим NTP.

```
R12(config)#clock timezone MSK 3
R12(config)#
*Oct 18 06:33:43.607: %SYS-6-CLOCKUPDATE: System clock has been updated from 06:33:43 UTC Sat Oct 18 2025 to 09:33:43 MSK Sat Oct 18 2025, configured from console by console.
R12(config)#
```

```
R13(config)#clock timezone MSK 3
R13(config)#
*Oct 18 06:33:43.607: %SYS-6-CLOCKUPDATE: System clock has been updated from 06:33:43 UTC Sat Oct 18 2025 to 09:33:43 MSK Sat Oct 18 2025, configured from console by console.
R13(config)#clock timezone MSK 3
R13(config)#
```

```
R15(config)#ntp server 2001:2025:1111:4::1
R15(config)#ntp update-calendar
R15(config)#do sh ntp status
Clock is synchronized, stratum 2, reference is 2001:2025:1111:4::1 nominal freq is 250.0000 Hz, actual freq is 250.0021 Hz, precision is 2**10
ntp uptime is 149800 (1/100 of seconds), resolution is 4000
reference time is EC9DC247.45E354B8 (11:37:43.273 MSK Sat Oct 18 2025)
clock offset is -0.5000 msec, root delay is 1.00 msec
root dispersion is 8.35 msec, peer dispersion is 4.44 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is -0.000008506 s/s
system poll interval is 64, last update was 17 sec ago.
```

```
R12(config)#clock timezone MSK 3
R12(config)#
R12(config)#
*Oct 18 07:47:04.634: %SYS-6-CLOCKUPDATE: System clock has been updated from 07:47:04 UTC Sat Oct 18 2025 to 10:47:04 MSK Sat Oct 18 2025, configured from console by console.
R12(config)#ntp master 1
```

```
R12(config)#clock timezone MSK 3
R12(config)#
*Oct 18 07:47:04.634: %SYS-6-CLOCKUPDATE: System clock has been updated from 07:47:04 UTC Sat Oct 18 2025 to 10:47:04 MSK Sat Oct 18 2025, configured from console by console.
R12(config)#ntp master 1
```

```
R14(config)#ntp server 2001:2025:1111:2::1
R14(config)#
*Oct 18 07:48:56.406: %SYS-6-CLOCKUPDATE: System clock has been updated from 07:48:56 UTC Sat Oct 18 2025 to 10:48:56 MSK Sat Oct 18 2025, configured from console by console.
```

Такую настройку проделываем на всех устройствах, не вижу смысла тут отображать ввод 2-х команд на оборудование ;)

При использовании IPv6 в локальных сетях отпадает необходимость в NAT так как адреса маршрутизируются в сети Интернет.