# Пректная работа.





Spine1
```sh
fabric forwarding anycast-gateway-mac 0000.0011.1234

route-map NH_UNCHANGED permit 10
  set ip next-hop unchanged
vrf context 1

interface Vlan1

interface Vlan35
  ip pim sparse-mode
  ip igmp static-oif 233.26.38.5

interface Vlan255
  ip address 172.16.255.44/24

interface Vlan701
  no shutdown
  ip address 192.168.70.5/30
  ip pim sparse-mode
  ip igmp join-group 233.26.38.5
  ip igmp join-group 233.26.38.6
  ip igmp join-group 233.26.38.13

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  member vni 1010
    ingress-replication protocol bgp
  member vni 10100 associate-vrf

interface Ethernet1/1
  description to_Life1
  no switchport
  ip address 10.0.0.0/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description to_Life2
  no switchport
  ip address 10.0.0.2/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/5
  description to_life3
  no switchport
  ip address 10.0.0.4/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface loopback0
  ip address 10.1.1.1/32
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
line console
line vty
boot nxos bootflash:/nxos.9.3.13.bin
router ospf 1
  router-id 10.1.1.1
router bgp 65000
  router-id 10.1.1.1
  address-family l2vpn evpn
    maximum-paths 10
    retain route-target all
  neighbor 10.0.1.1
    remote-as 65001
    update-source loopback0
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map NH_UNCHANGED out
      rewrite-evpn-rt-asn
  neighbor 10.0.1.2
    remote-as 65002
    update-source loopback0
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map NH_UNCHANGED out
      rewrite-evpn-rt-asn
  neighbor 10.0.1.3
    remote-as 65003
    update-source loopback0
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map NH_UNCHANGED out
      rewrite-evpn-rt-asn
```

Конфигурация Spine2 аналогичная.


Life1
```sh
nv overlay evpn
feature ospf
feature bgp
feature pim
feature eigrp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature vpc
feature nv overlay

fabric forwarding anycast-gateway-mac 000e.000e.000e
vlan 1,10,20,999
vlan 10
  name VLAN_10
  vn-segment 10010
vlan 20
  name VLAN_20
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
  ip route 0.0.0.0/0 172.16.255.1
vrf context vpc_keepalive
vpc domain 7
  peer-switch
  role priority 100
  peer-keepalive destination 1.0.0.2 source 1.0.0.1 vrf vpc_keepalive
  delay restore 150
  peer-gateway
  ip arp synchronize

interface Vlan1
  no ip redirects
  no ipv6 redirects

interface Vlan10
  no shutdown
  vrf member A
  no ip redirects
  ip address 172.16.10.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member A
  no ip redirects
  ip address 172.16.20.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan999
  no shutdown
  vrf member A
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  description Client
  switchport mode trunk
  switchport trunk allowed vlan 10
  spanning-tree port type network
  vpc peer-link

interface port-channel10
  switchport mode trunk
  switchport trunk allowed vlan 10
  vpc 10

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
  ip address 10.0.0.1/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description to_Spine2
  no switchport
  ip address 10.0.0.11/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/10
  description Client
  switchport mode trunk
  switchport trunk allowed vlan 10
  channel-group 10 mode active

interface Ethernet1/44
  description vpc_keepalive
  no switchport
  speed 1000
  vrf member vpc_keepalive
  ip address 1.0.0.1/30
  no shutdown

interface Ethernet1/45
  switchport mode trunk
  switchport trunk allowed vlan 10
  channel-group 1 mode active

interface Ethernet1/46
  switchport mode trunk
  switchport trunk allowed vlan 10
  channel-group 1 mode active


interface loopback1
  ip address 10.0.1.1/32
  ip address 10.0.1.254/32 secondary
  ip router ospf 1 area 0.0.0.0

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
  neighbor 10.1.1.1
    inherit peer SPINES
  neighbor 10.1.1.2
    inherit peer SPINES
evpn
  vni 10010 l2
    rd 65001:10010
    route-target import 65000:10010
    route-target export 65000:10010
  vni 10020 l2
    rd 65001:10020
    route-target import 65000:10020
    route-target export 65000:10020
```


Life2
```sh
nv overlay evpn
feature ospf
feature bgp
feature pim
feature eigrp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature vpc
feature nv overlay

fabric forwarding anycast-gateway-mac 000e.000e.000e
vlan 1,10,20,999
vlan 10
  name VLAN_10
  vn-segment 10010
vlan 20
  name VLAN_20
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
  ip route 0.0.0.0/0 10.0.201.1
vrf context vpc_keepalive
vpc domain 7
  peer-switch
  role priority 100
  peer-keepalive destination 1.0.0.1 source 1.0.0.2 vrf vpc_keepalive
  delay restore 150
  peer-gateway
  ip arp synchronize

interface Vlan1
  no ip redirects
  no ipv6 redirects

interface Vlan10
  no shutdown
  vrf member A
  no ip redirects
  ip address 172.16.10.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member A
  no ip redirects
  ip address 172.16.20.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan999
  no shutdown
  vrf member A
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  description Client
  switchport mode trunk
  switchport trunk allowed vlan 10
  spanning-tree port type network
  vpc peer-link

interface port-channel10
  switchport mode trunk
  switchport trunk allowed vlan 10
  vpc 10

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
  ip address 10.0.0.3/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description to_Spine2
  no switchport
  ip address 10.0.0.9/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/10
  description Client
  switchport mode trunk
  switchport trunk allowed vlan 10
  channel-group 10 mode active

interface Ethernet1/44
  description vpc_keepalive
  no switchport
  speed 1000
  vrf member vpc_keepalive
  ip address 1.0.0.2/30
  no shutdown

interface Ethernet1/45
  switchport mode trunk
  switchport trunk allowed vlan 10
  channel-group 1 mode active

interface Ethernet1/46
  switchport mode trunk
  switchport trunk allowed vlan 10
  channel-group 1 mode active

interface Ethernet1/47
  switchport access vlan 10


interface loopback1
  ip address 10.0.1.2/32
  ip address 10.0.1.254/32 secondary
  ip router ospf 1 area 0.0.0.0

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
Конфигурация второй пары лифов аналогичная.

BorderLife
```sh
fabric forwarding anycast-gateway-mac 000e.000e.000e
vlan 1,10,20,999
vlan 10
  name VLAN_10
  vn-segment 10010
vlan 20
  name VLAN_20
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

interface Ethernet1/3
  no switchport
  ip address 10.0.0.7/31
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/47
  no switchport
  vrf member A
  ip address 192.168.10.0/31
  no shutdown

interface loopback1
  ip address 10.0.1.3/32
  ip router ospf 1 area 0.0.0.0

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
  vrf A
    address-family ipv4 unicast
      network 172.16.10.0/24
      network 172.16.20.0/24
    neighbor 192.168.10.1
      remote-as 65100
      address-family ipv4 unicast
        next-hop-self
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



Конфигурация маршрутизатора
```sh
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.254
 duplex auto
 speed auto
 media-type rj45
 negotiation auto
end

router bgp 65100
 bgp log-neighbor-changes
 neighbor 192.168.10.0 remote-as 65003
 !
 address-family ipv4
  neighbor 192.168.10.0 activate
  neighbor 192.168.10.0 next-hop-self
  neighbor 192.168.10.0 allowas-in 1
  no auto-summary
  no synchronization
  network 0.0.0.0
```

Таким образом через маршрутизатор будет производиться общение с хостами вне фабрики.

Остальось только проверить работоспособность:


