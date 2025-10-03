# VXLAN. Multihoming

### Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming.


Топология так же неизменна, 3 лифа и два спйна.

К 1 и 2 лифам будет подключен клиент к 3 лифу будет подключен второй клиент.
 Клиентов посадил в одну сеть.

Приступим к натройке:

Life1
```sh
fabric forwarding anycast-gateway-mac 000e.000e.000e
vlan 1,10,20,999
vlan 10
  name VLAN_10
  vn-segment 10010
vlan 20
  vn-segment 10020
vlan 999
  vn-segment 100999

vrf context A
  vni 100999
  rd 65002:100999
  address-family ipv4 unicast
    route-target import 65000:100999
    route-target import 65000:100999 evpn
    route-target export 65000:100999
    route-target export 65000:100999 evpn
vrf context management
  ip route 0.0.0.0/0 10.0.1.1
  ip route 0.0.0.0/0 172.16.255.1

interface Vlan1

interface Vlan10
  no shutdown
  vrf member A
  ip address 172.16.10.254/24
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member A
  ip address 172.16.20.254/24
  fabric forwarding mode anycast-gateway

interface Vlan999
  no shutdown
  vrf member A
  ip forward

interface port-channel1
  description Client
  switchport mode trunk
  switchport trunk allowed vlan 10
  ethernet-segment 1
    system-mac 0000.0000.0001

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
  member vni 100999 associate-vrf

interface Ethernet1/1
  description to_Spine1
  no switchport
  evpn multihoming core-tracking
  ip address 10.0.0.1/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description to_Spine2
  no switchport
  evpn multihoming core-tracking
  ip address 10.0.0.11/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/10
  description Client
  switchport mode trunk
  switchport trunk allowed vlan 10
  channel-group 1 mode active

interface loopback1
  ip address 10.0.1.1/32
  ip router ospf 1 area 0.0.0.0
line console
line vty
boot nxos bootflash:/nxos.9.3.14.bin
router ospf 1
  router-id 10.0.1.1
router bgp 65001
  router-id 10.0.1.1
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10
  template peer SPINES
    remote-as 65000
    update-source loopback1
    ebgp-multihop 10
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  neighbor 10.1.1.1
    inherit peer SPINES
  neighbor 10.1.1.2
    inherit peer SPINES
evpn
  vni 10010 l2
    rd 65002:10010
    route-target import auto
    route-target import 65000:10010
    route-target export auto
    route-target export 65000:10010
  vni 10020 l2
    rd 65002:10020
    route-target import 65000:10020
```
Life2
```sh 
fabric forwarding anycast-gateway-mac 000e.000e.000e
vlan 1,10,20,999
vlan 10
  name VLAN_10
  vn-segment 10010
vlan 20
  vn-segment 10020
vlan 999
  vn-segment 100999

vrf context A
  vni 100999
  rd 65002:100999
  address-family ipv4 unicast
    route-target import 65000:100999
    route-target import 65000:100999 evpn
    route-target export 65000:100999
    route-target export 65000:100999 evpn

interface Vlan1

interface Vlan10
  no shutdown
  vrf member A
  ip address 172.16.10.254/24
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member A
  ip address 172.16.20.254/24
  fabric forwarding mode anycast-gateway

interface Vlan999
  no shutdown
  vrf member A
  ip forward

interface port-channel1
  description Client
  switchport mode trunk
  switchport trunk allowed vlan 10
  ethernet-segment 1
    system-mac 0000.0000.0001

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
  member vni 100999 associate-vrf

interface Ethernet1/1
  description to_Spine1
  no switchport
  evpn multihoming core-tracking
  ip address 10.0.0.3/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description to_Spine2
  no switchport
  evpn multihoming core-tracking
  ip address 10.0.0.9/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/10
  description Client
  switchport mode trunk
  switchport trunk allowed vlan 10
  channel-group 1 mode active

interface loopback1
  ip address 10.0.1.2/32
  ip router ospf 1 area 0.0.0.0
line console
line vty
boot nxos bootflash:/nxos.9.3.14.bin
router ospf 1
  router-id 10.0.1.2
router bgp 65002
  router-id 10.0.1.2
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10
  template peer SPINES
    remote-as 65000
    update-source loopback1
    ebgp-multihop 10
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.1.1.1
    inherit peer SPINES
  neighbor 10.1.1.2
    inherit peer SPINES
evpn
  vni 10010 l2
    rd 65002:10010
    route-target import 65000:10010
    route-target export 65000:10010
  vni 10020 l2
    rd 65002:10020
    route-target import 65000:10020
    route-target export 65000:10020
```

Life3
```sh 
fabric forwarding anycast-gateway-mac 000e.000e.000e
vlan 1,10,20,999
vlan 10
  name VLAN_10
  vn-segment 10010
vlan 20
  vn-segment 10020
vlan 999
  vn-segment 100999

vrf context A
  vni 100999
  rd 65003:100999
  address-family ipv4 unicast
    route-target import 65000:100999
    route-target import 65000:100999 evpn
    route-target export 65000:100999
    route-target export 65000:100999 evpn

interface Vlan1

interface Vlan10
  no shutdown
  vrf member A
  ip address 172.16.10.254/24
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member A
  ip address 172.16.20.254/24
  fabric forwarding mode anycast-gateway

interface Vlan999
  no shutdown
  vrf member A
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
  member vni 100999 associate-vrf

interface Ethernet1/1
  no switchport
  ip address 10.0.0.5/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/2

interface Ethernet1/3
  no switchport
  ip address 10.0.0.7/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/47
  switchport access vlan 10

interface loopback1
  ip address 10.0.1.3/32
  ip router ospf 1 area 0.0.0.0
line console
line vty
boot nxos bootflash:/nxos.9.3.14.bin
router ospf 1
  router-id 10.0.1.3
router bgp 65003
  router-id 10.0.1.3
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10
  template peer SPINES
    remote-as 65000
    update-source loopback1
    ebgp-multihop 10
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.1.1.1
    inherit peer SPINES
  neighbor 10.1.1.2
    inherit peer SPINES
evpn
  vni 10010 l2
    rd 65003:10010
    route-target import 65000:10010
    route-target export 65000:10010
  vni 10020 l2
    rd 65003:10020
    route-target import 65000:10020
    route-target export 65000:10020
```


Проврка работоспособности:

![alt text](image.png)

![alt text](image-1.png)

![alt text](image-2.png)

![alt text](image-3.png)


```sh 
Life3# sh bg l2 e
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 104, Local Router ID is 10.0.1.3
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 65002:10010
* e[1]:[0300.0000.0000.0100.0001]:[0x0]/152
                      10.0.1.1                                       0 65000 650
01 i
*>e                   10.0.1.1                                       0 65000 650
01 i
* e[2]:[0]:[0]:[48]:[001c.7363.321f]:[0]:[0.0.0.0]/216
                      10.0.1.2                                       0 65000 650
02 i
*>e                   10.0.1.2                                       0 65000 650
02 i
* e[2]:[0]:[0]:[48]:[001c.7363.321f]:[32]:[172.16.10.50]/272
                      10.0.1.1                                       0 65000 650
01 i
*>e                   10.0.1.1                                       0 65000 650
01 i
* e[3]:[0]:[32]:[10.0.1.1]/88
                      10.0.1.1                                       0 65000 650
01 i
*>e                   10.0.1.1                                       0 65000 650
01 i
* e[3]:[0]:[32]:[10.0.1.2]/88
                      10.0.1.2                                       0 65000 650
02 i
*>e                   10.0.1.2                                       0 65000 650
02 i

Route Distinguisher: 65002:10020
* e[3]:[0]:[32]:[10.0.1.1]/88
                      10.0.1.1                                       0 65000 650
01 i
*>e                   10.0.1.1                                       0 65000 650
01 i
* e[3]:[0]:[32]:[10.0.1.2]/88
                      10.0.1.2                                       0 65000 650
02 i
*>e                   10.0.1.2                                       0 65000 650
02 i

Route Distinguisher: 65003:10010    (L2VNI 10010)
*>e[1]:[0300.0000.0000.0100.0001]:[0x0]/152
                      10.0.1.1                                       0 65000 650
01 i
*>e[2]:[0]:[0]:[48]:[001c.7363.321f]:[0]:[0.0.0.0]/216
                      10.0.1.2                                       0 65000 650
02 i
*>l[2]:[0]:[0]:[48]:[5475.d06a.011b]:[0]:[0.0.0.0]/216
                      10.0.1.3                          100      32768 i
*>e[2]:[0]:[0]:[48]:[001c.7363.321f]:[32]:[172.16.10.50]/272
                      10.0.1.1                                       0 65000 650
01 i
*>l[2]:[0]:[0]:[48]:[5475.d06a.011b]:[32]:[172.16.10.10]/272
                      10.0.1.3                          100      32768 i
*>e[3]:[0]:[32]:[10.0.1.1]/88
                      10.0.1.1                                       0 65000 650
01 i
*>e[3]:[0]:[32]:[10.0.1.2]/88
                      10.0.1.2                                       0 65000 650
02 i
*>l[3]:[0]:[32]:[10.0.1.3]/88
                      10.0.1.3                          100      32768 i

Route Distinguisher: 65003:10020    (L2VNI 10020)
*>e[3]:[0]:[32]:[10.0.1.1]/88
                      10.0.1.1                                       0 65000 650
01 i
*>e[3]:[0]:[32]:[10.0.1.2]/88
                      10.0.1.2                                       0 65000 650
02 i
*>l[3]:[0]:[32]:[10.0.1.3]/88
                      10.0.1.3                          100      32768 i

Route Distinguisher: 10.0.1.1:40
* e[1]:[0300.0000.0000.0100.0001]:[0xffffffff]/152
                      10.0.1.1                                       0 65000 650
01 i
*>e                   10.0.1.1                                       0 65000 650
01 i

Route Distinguisher: 10.0.1.2:40
* e[1]:[0300.0000.0000.0100.0001]:[0xffffffff]/152
                      10.0.1.2                                       0 65000 650
02 i
*>e                   10.0.1.2                                       0 65000 650
02 i


Route Distinguisher: 10.0.1.3:65534    (L2VNI 0)
* e[1]:[0300.0000.0000.0100.0001]:[0xffffffff]/152
                      10.0.1.2                                       0 65000 650
02 i
*>e                   10.0.1.1                                       0 65000 650
01 i

Route Distinguisher: 65003:100999    (L3VNI 100999)
*>e[2]:[0]:[0]:[48]:[001c.7363.321f]:[32]:[172.16.10.50]/272
                      10.0.1.1                                       0 65000 650
01 i

```



Пускал пинг от клюента подключенному от лифа 3 до клиента который подключен с помощю мульихоминга.

Отключил сначала один линк к клиенту, после чего произошло переключение на другой видимо и потерялось 2 пакета.
После включения я сразу же погасил второй, здесь удже потерялось около 15 пакетов.

Статистика следующая:
Success rate is 99 percent (247719/247736), round-trip min/avg/max = 1/1/12 ms

Цели считаю достигнутыми, мультихоминг отрабатывает как надо.

Еще бросилос в глаза что при первом пинге от клиента подключенному по LACP в ответ прилетали пакеты в месте с дубликатами.

![alt text](image-4.png)


После, такого замеченно не было.








