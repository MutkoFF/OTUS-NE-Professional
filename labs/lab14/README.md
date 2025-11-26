## IPSec over DmVPN

### Цели:
1) Настроите GRE поверх IPSec между офисами Москва и С.-Петербург;
2) Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги;
3) Все узлы в офисах в лабораторной работе должны иметь IP связность;
4) План работы и изменения зафиксированы в документации.

Для начала настроим PKI-сервер на R15 в Москве

```
R15(config)#crypto pki server MY-CS
R15(cs-server)# database url  unix:/cerbase
R15(cs-server)# database level complete
R15(cs-server)# issuer-name CN=MY-CS,OU=VPN,O=Cisco,C=RU
R15(cs-server)# hash sha256
R15(cs-server)# lifetime certificate 730
R15(cs-server)# lifetime ca-certificate 1000
R15(cs-server)# lifetime crl 336
R15(cs-server)# cdp-url http://[2001:2025:abcd:ff03::]/cerbase/MY-CS.crl
R15(cs-server)# no shut
%Some server settings cannot be changed after CA certificate generation.
% Please enter a passphrase to protect the private key
% or type Return to exit
Password:

Re-enter password:
% Generating 1024 bit RSA keys, keys will be non-exportable...
[OK] (elapsed time was 1 seconds)

% Certificate Server enabled.
R15(cs-server)#
*Nov 23 13:22:13.343: %PKI-6-CS_ENABLED: Certificate server now enabled.
R15(cs-server)#
R15(cs-server)#
R15(cs-server)#do show crypto pki server MY-CS
Certificate Server MY-CS:
    Status: enabled
    State: enabled
    Server's configuration is locked  (enter "shut" to unlock it)
    Issuer name: CN=MY-CS,OU=VPN,O=Cisco,C=RU
    CA cert fingerprint: 3BB7D24C DC6F722D 135764C7 912FB26F
    Granting mode is: manual
    Last certificate issued serial number (hex): 1
    CA certificate expiration timer: 18:18:32 MSK Aug 19 2028
    CRL NextUpdate timer: 17:18:32 MSK Dec 7 2025
    Current primary storage dir: unix:/cerbase
    Database Level: Complete - all issued certs written as <serialnum>.cer
R15(cs-server)#
R15(cs-server)#do show crypto pki server MY-CS cert
Serial Issued date              Expire date               Subject Name
1       17:18:32 MSK Nov 23 2025 18:18:32 MSK Aug 19 2028  cn=MY-CS ou=VPN o=Cisco c=RU
```
Далее, на R15 сконфигурируем IPsec и GRE
```
R15(config)# crypto ikev2 proposal GRE-PROP
R15(config-ikev2-proposal)# encryption aes-cbc-256
R15(config-ikev2-proposal)# integrity sha256
R15(config-ikev2-proposal)# group 14
R15(config-ikev2-proposal)# exit
R15(config)# crypto ikev2 policy GRE-POL
R15(config-ikev2-policy)# proposal GRE-PROP
R15(config-ikev2-profile)# exit

R15(config)# crypto ikev2 profile GRE-PROF
R15(config-ikev2-profile)# match identity remote fqdn R18
R15(config-ikev2-profile)# identity local fqdn R15
R15(config-ikev2-profile)# authentication local rsa-sig
R15(config-ikev2-profile)# authentication remote rsa-sig
R15(config-ikev2-profile)# pki trustpoint MY-CA
R15(config-ikev2-profile)# exit

R15(config)# crypto ipsec transform-set GRE-SET esp-aes 256 esp-sha256-hmac
R15(config)# crypto ipsec profile GRE-IPSEC
R15(config-ipsec-profile)# set transform-set GRE-SET
R15(config-ipsec-profile)# set ikev2-profile GRE-PROF
R15(config-ikev2-profile)# exit

R15(config)# interface Tunnel2
R15(config-if)# ipv6 address 2001:2025:5555::1/127
R15(config-if)# tunnel source Ethernet0/2
R15(config-if)# tunnel destination 2001:2025:ABCD:FF05::
R15(config-if)# tunnel mode gre ipv6
R15(config-if)# tunnel protection ipsec profile GRE-IPSEC
R15(config-if)# end

R15# show crypto ikev2 sa
IPv6 Crypto IKEv2 SAs

Session-id:1, Status:UP-ACTIVE
  Authentication: RSA-SIG
  Local  FQDN: R15
  Remote FQDN: R18
  Encr: AES-CBC-256  Hash: SHA256  DH Grp:14

R15# show crypto ipsec sa
interface: Tunnel2
  local  addr  2001:2025:ABCD:FF03::
  remote addr  2001:2025:ABCD:FF05::
  #pkts encaps: 38
  #pkts decaps: 37

```

После этого, на R18 сконфигурируем IPsec и GRE

```
R18(config)# crypto ikev2 proposal GRE-PROP
R18(config-ikev2-proposal)# encryption aes-cbc-256
R18(config-ikev2-proposal)# integrity sha256
R18(config-ikev2-proposal)# group 14
R18(config-ikev2-proposal)# exit

R18(config)# crypto ikev2 policy GRE-POL
R18(config-ikev2-policy)# proposal GRE-PROP
R18(config-ikev2-policy)# exit

R18(config)# crypto ikev2 profile GRE-PROF
R18(config-ikev2-profile)# match identity remote fqdn R15
R18(config-ikev2-profile)# identity local fqdn R18
R18(config-ikev2-profile)# authentication local rsa-sig
R18(config-ikev2-profile)# authentication remote rsa-sig
R18(config-ikev2-profile)# pki trustpoint MY-CA
R18(config-ikev2-profile)# exit

R18(config)# crypto ipsec transform-set GRE-SET esp-aes 256 esp-sha256-hmac
R18(config)# crypto ipsec profile GRE-IPSEC
R18(config-ipsec-profile)# set transform-set GRE-SET
R18(config-ipsec-profile)# set ikev2-profile GRE-PROF
R18(config-ipsec-profile)# exit

R18(config)# interface Tunnel2
R18(config-if)# ipv6 address 2001:2025:5555::/127
R18(config-if)# tunnel source Ethernet0/2
R18(config-if)# tunnel destination 2001:2025:ABCD:FF03::
R18(config-if)# tunnel mode gre ipv6
R18(config-if)# tunnel protection ipsec profile GRE-IPSEC
R18(config-if)# end
```
Затем, проверим работоспособность
```
R18# show crypto ikev2 sa
IPv6 Crypto IKEv2 SAs

Session-id:2, Status:UP-ACTIVE
  Peer: 2001:2025:ABCD:FF03::<500>
  Local: 2001:2025:ABCD:FF05::<500>
  
  Authentication: RSA-SIG
  Local  FQDN: R18
  Remote FQDN: R15

  Encr: AES-CBC-256
  Integrity: SHA256
  DH Grp:14

  IKEv2 SA: established 00:03:12 ago
  Rekeying: no

R18# show interface Tunnel2
Tunnel2 is up, line protocol is up
  Hardware is Tunnel
  IPv6 address 2001:2025:5555::/127
  MTU 1476 bytes, BW 100 Kbit
  Tunnel source 2001:2025:ABCD:FF05::, destination 2001:2025:ABCD:FF03::
  Tunnel protocol/transport GRE/IP
  Tunnel protection via IPsec profile: GRE-IPSEC
  5 minute input rate 2000 bits/sec, 3 packets/sec
  5 minute output rate 2000 bits/sec, 3 packets/sec

R18# ping ipv6 2001:2025:5555::1 source 2001:2025:5555::
Type escape sequence to abort.
Sending 5, 16-byte ICMP Echos to 2001:2025:5555::1, timeout is 2 seconds:
Packet sent with a source address of 2001:2025:5555::
!!!!!
Success rate is 100 percent (5/5)
```

Далее, уже на R14 развернём PKI-сервер для дальнейшего использования в DMVPN

```
R14(config)#crypto pki server MY-CS
R14(cs-server)#database url  unix:/cerbase
% Server database url was changed. You need to move the
% existing database to the new location.
R14(cs-server)#database level complete
R14(cs-server)#issuer-name CN=MY-CS,OU=VPN,O=Cisco,C=RU
R14(cs-server)#hash sha256
R14(cs-server)#lifetime certificate 730
R14(cs-server)#lifetime ca-certificate 1000
R14(cs-server)#lifetime crl 336
R14(cs-server)#cdp-url http://[2001:2025:ABCD:FF00::]/cerbase/MY-CS.crl
R14(cs-server)#no shut
%Some server settings cannot be changed after CA certificate generation.
% Please enter a passphrase to protect the private key
% or type Return to exit
Password:

Re-enter password:
% Generating 1024 bit RSA keys, keys will be non-exportable...
[OK] (elapsed time was 1 seconds)

R14(cs-server)#
*Nov 26 01:34:18.274: %SSH-5-ENABLED: SSH 1.99 has been enabled
R14(cs-server)#do show crypto pki server MY-CS
Certificate Server MY-CS:
    Status: disabled, HTTP Server is disabled
    State: check failed
    Server's configuration is locked  (enter "shut" to unlock it)
    Issuer name: CN=MY-CS,OU=VPN,O=Cisco,C=RU
    CA cert fingerprint: D6C64E49 FB5987E4 65D35336 D0C3FA5A
    Granting mode is: manual
    Last certificate issued serial number (hex): 1
    CA certificate expiration timer: 04:34:18 MSK Aug 22 2028
    CRL NextUpdate timer: 04:34:18 MSK Dec 10 2025
    Current primary storage dir: unix:/cerbase
    Database Level: Complete - all issued certs written as <serialnum>.cer
```

После этого переходим к конфигурированию IPsec и DMVPN
```
R14(config)#crypto ikev2 proposal DMVPN-PROP
R14(config-ikev2-proposal)#encryption aes-cbc-256
R14(config-ikev2-proposal)#integrity sha256
R14(config-ikev2-proposal)#group 14
R14(config-ikev2-proposal)#exit

R14(config)#crypto ikev2 policy DMVPN-POLICY
R14(config-ikev2-policy)#proposal DMVPN-PROP
R14(config-ikev2-policy)#exit

R14(config)#crypto ikev2 profile DMVPN-IKEV2
R14(config-ikev2-profile)#match identity remote fqdn vpn
R14(config-ikev2-profile)#authentication remote rsa-sig
R14(config-ikev2-profile)#authentication local rsa-sig
R14(config-ikev2-profile)#pkcs1
R14(config-ikev2-profile)#identity local fqdn R14
R14(config-ikev2-profile)#trustpoint MY-CS-TP
R14(config-ikev2-profile)#exit

R14(config)#crypto ipsec transform-set DMVPN-SET esp-aes 256 esp-sha256-hmac
R14(cfg-crypto-trans)#mode transport
R14(cfg-crypto-trans)#exit

R14(config)#crypto ipsec profile DMVPN-IPv6
R14(config-ipsec-profile)#set transform-set DMVPN-SET
R14(config-ipsec-profile)#set ikev2-profile DMVPN-IKEV2

R14(config)#interface Tunnel2
R14(config-if)#ipv6 address 2001:2025:6666::1/128
R14(config-if)#tunnel source Ethernet0/2
R14(config-if)#tunnel mode gre multipoint
R14(config-if)#ipv6 mtu 1400
R14(config-if)#ipv6 ospf6 network broadcast
R14(config-if)#tunnel protection ipsec profile DMVPN-IPv6
```

Далее, на R27 запросим сертификаты и настроим DMVPN
```
R27(config)#crypto pki trustpoint MY-CS-TP
R27(ca-trustpoint)#enrollment url http://[2001:2025:ABCD:FF00::]
R27(ca-trustpoint)#subject-name CN=R27,OU=VPN,O=Cisco,C=RU
R27(ca-trustpoint)#revocation-check crl
R27(ca-trustpoint)#rsakeypair R27-KEY
R27(ca-trustpoint)#exit
R27(config)#crypto pki enroll MY-CS-TP

R27(config)#crypto ikev2 proposal DMVPN-PROP
R27(config-ikev2-proposal)#encryption aes-cbc-256
R27(config-ikev2-proposal)#integrity sha256
R27(config-ikev2-proposal)#group 14
R27(config-ikev2-proposal)#exit

R27(config)#crypto ikev2 policy DMVPN-POLICY
R27(config-ikev2-policy)#proposal DMVPN-PROP
R27(config-ikev2-policy)#exit

R27(config)#crypto ikev2 profile DMVPN-IKEV2
R27(config-ikev2-profile)#match identity remote fqdn R14
R27(config-ikev2-profile)#authentication remote rsa-sig
R27(config-ikev2-profile)#authentication local rsa-sig
R27(config-ikev2-profile)#identity local fqdn R27
R27(config-ikev2-profile)#trustpoint MY-CS-TP

R27(config)#crypto ipsec transform-set DMVPN-SET esp-aes 256 esp-sha256-hmac
R27(cfg-crypto-trans)#mode transport

R27(config)#crypto ipsec profile DMVPN-IPv6
R27(config-ipsec-profile)#set transform-set DMVPN-SET
R27(config-ipsec-profile)#set ikev2-profile DMVPN-IKEV2

R27(config)#interface Tunnel2
R27(config-if)#ipv6 address 2001:2025:6666::2/128
R27(config-if)#tunnel source Ethernet0/0
R27(config-if)#tunnel mode gre multipoint
R27(config-if)#tunnel destination 2001:2025:FFEE::1
R27(config-if)#tunnel protection ipsec profile DMVPN-IPv6
```

Затем, на R28 запросим сертификаты и настроим DMVPN
```
R28(config)#crypto pki trustpoint MY-CS-TP
R28(ca-trustpoint)#enrollment url http://[2001:2025:ABCD:FF00::]
R28(ca-trustpoint)#subject-name CN=R27,OU=VPN,O=Cisco,C=RU
R28(ca-trustpoint)#revocation-check crl
R28(ca-trustpoint)#rsakeypair R28-KEY
R28(ca-trustpoint)#exit
R28(config)#crypto pki enroll MY-CS-TP

R28(config)#crypto ikev2 proposal DMVPN-PROP
R28(config-ikev2-proposal)#encryption aes-cbc-256
R28(config-ikev2-proposal)#integrity sha256
R28(config-ikev2-proposal)#group 14
R28(config-ikev2-proposal)#exit

R28(config)#crypto ikev2 policy DMVPN-POLICY
R28(config-ikev2-policy)#proposal DMVPN-PROP
R28(config-ikev2-policy)#exit

R28(config)#crypto ikev2 profile DMVPN-IKEV2
R28(config-ikev2-profile)#match identity remote fqdn R14
R28(config-ikev2-profile)#authentication remote rsa-sig
R28(config-ikev2-profile)#authentication local rsa-sig
R28(config-ikev2-profile)#identity local fqdn R27
R28(config-ikev2-profile)#trustpoint MY-CS-TP

R28(config)#crypto ipsec transform-set DMVPN-SET esp-aes 256 esp-sha256-hmac
R28(cfg-crypto-trans)#mode transport

R28(config)#crypto ipsec profile DMVPN-IPv6
R28(config-ipsec-profile)#set transform-set DMVPN-SET
R27(config-ipsec-profile)#set ikev2-profile DMVPN-IKEV2

R28(config)#interface Tunnel2
R28(config-if)#ipv6 address 2001:2025:6666::3/128
R28(config-if)#tunnel source Ethernet0/1
R28(config-if)#tunnel mode gre multipoint
R28(config-if)#tunnel destination 2001:2025:FFEE::1
R28(config-if)#tunnel protection ipsec profile DMVPN-IPv6
```