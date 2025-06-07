## VLAN

### Цели:
1) Построить топологию Router-on-a-Stick;
2) Настроить маршрутизацию трафика между VLAN'ами.

### Исходная топология:

![Картинка](./pictures/lab01-topology.jpeg)

**Шаг 1. Выполним базовую настройку роутера**
```cli
Router>en
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R1
R1(config)#no ip doma
R1(config)#no ip domain-lookup
R1(config)#enable secert class
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#
R1(config-line)#pass cisco
R1(config-line)#login
R1(config-line)#exit
R1(config)#line vty 0 4
R1(config-line)#pass cisco
R1(config-line)#login
R1(config-line)#exit
R1(config)#service password-encryption
R1(config)#banner motd $ Unauthorized access is prohibited! $
R1#wr
Building configuration...
[OK]
R1#
R1#clock set 19:14:00 june 07 2025
R1#sh clock
19:14:3.922 UTC Sat Jun 7 2025
```