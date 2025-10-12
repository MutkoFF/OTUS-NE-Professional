## BGP. Фильтрация

### Цели:
1) Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path); 
2) Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list);
3) Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию;
4) Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург;
5) Все сети в лабораторной работе должны иметь IP связность.

Для начала, настроим фильтрацию транзитного трафика в Москве:

Настройка соседства BGP на маршрутизаторе R14
```
R14(config)#ip as-path access-list 1 deny _1001_
R14(config)#ip as-path access-list 1 permit .*
R14(config)#router bgp 1001
R14(config-router)#address-family ipv6
R14(config-router-af)# neighbor 2001:2025:ABCD:FF00::1 filter-list 1 in
```

Настройка соседства BGP на маршрутизаторе R15
```
R15(config)#ip as-path access-list 1 deny _1001_
R15(config)#ip as-path access-list 1 permit .*
R15(config)#router bgp 1001
R15(config-router)#address-family ipv6
R15(config-router-af)#neighbor 2001:2025:ABCD:FF03::1 filter-list 1 in
```

Далее, настроим роутер R18 в Санкт-Петербурге так, чтобы в сторону провайдеров он отдавал только префикс своих внутренних сетей
```
R18(config)#ipv6 prefix-list SPB-PREFIXES seq 5 permit 2001:2025:3333::/48 le 64
R18(config)#router bgp 2042
R18(config-router)# address-family ipv6
R18(config-router-af)#neighbor 2001:2025:ABCD:FF05::1 prefix-list SPB-PREFIXES out
R18(config-router-af)#neighbor 2001:2025:ABCD:FF09::1 prefix-list SPB-PREFIXES out
```

Затем, настроим роутер R22 так, чтобы в офис Москва отдавался только маршрут по умолчанию
```
R22(config)#ipv6 prefix-list ONLY-DEFAULT seq 5 permit ::/0
R22(config)#ipv6 prefix-list ONLY-DEFAULT seq 10 deny 0::/0 le 128
R22(config)#router bgp 101
R22(config-router)#address-family ipv6
R22(config-router-af)#neighbor 2001:2025:ABCD:FF00:: prefix-list ONLY-DEFAULT out
```

После этого, настроим роутер R21 так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербурга
```
R21(config)#ipv6 prefix-list DEFAULT-AND-SPB seq 5 permit ::/0
R21(config)#ipv6 prefix-list DEFAULT-AND-SPB seq 10 permit 2001:2025:3333::/48 le 64
R21(config)#ipv6 prefix-list DEFAULT-AND-SPB seq 15 deny 0::/0 le 128
R21(config)#router bgp 301
R21(config-router)# address-family ipv6
neighbor 2001:2025:ABCD:FF03:: prefix-list DEFAULT-AND-SPB out
```