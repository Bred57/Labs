# VxLAN. L2 VNI

### Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.

В качестве underlay будем использовать протокл OSPF.

Топология неизменна:
![alt text](image.png)

Добавил несколько клиентов.

Будем обмениться информацией из vlan 10, клиентская сеть будет использоваться 172.16.10.0/24

Приступаем к настройке: 

Конфигурация спайнов:
```sh
route-map NH_UNCHANGED permit 10
  set ip next-hop unchanged

interface Ethernet1/1
  no switchport
  ip address 10.10.0.1/31
  ip router isis 10
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.10.0.5/31
  ip router isis 10
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  no switchport
  ip address 10.10.0.9/31
  ip router isis 10
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface loopback1
  ip address 10.0.1.1/32
  ip router isis 10
  ip router ospf 1 area 0.0.0.0

router ospf 1
  router-id 10.0.1.1

router bgp 65000
  router-id 10.0.1.1
  address-family l2vpn evpn
    maximum-paths 10
    retain route-target all
  neighbor 10.1.0.1
    remote-as 65001
    update-source loopback1
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map NH_UNCHANGED out
      rewrite-evpn-rt-asn
  neighbor 10.1.0.2
    remote-as 65002
    update-source loopback1
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map NH_UNCHANGED out
      rewrite-evpn-rt-asn
  neighbor 10.1.0.3
    remote-as 65003
    update-source loopback1
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map NH_UNCHANGED out
      rewrite-evpn-rt-asn
```

Конфигурация лифов:
```sh
vlan 10
  name VLAN_10
  vn-segment 10010

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp

interface Ethernet1/1
  no switchport
  ip address 10.10.0.0/31
  ip router isis 10
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.10.0.2/31
  ip router isis 10
  ip ospf network point-to-point
  ip router ospf 1 area 0.0.0.0
  no shutdown

interface loopback1
  ip address 10.1.0.1/32
  ip router ospf 1 area 0.0.0.0

router ospf 1
  router-id 10.0.0.1

router bgp 65001
  router-id 10.0.0.1
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
  neighbor 10.0.1.1
    inherit peer SPINES
  neighbor 10.0.1.2
    inherit peer SPINES
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
```

Но здесь возникают проблемы, соседство поднимается и видно что spine принимают префиксы, но к сожаленияю, life не принимают от ставнов ничего:

![alt text](image-1.png)

В то время как спан от лифов получает:
![alt text](image-2.png)

Как следствие, пинга между клиентами нет: 
![alt text](image-3.png)



