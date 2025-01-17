## Дз6 Построение Vxlan EVPN для L3
### План работ:
1. Добавить L3 схему сети
2. Выделить IP адрес для ПК из другой сети 
3. Настройка VTEP на Leaf 2
4. Настроить Vlan 200 на Leaf 2 и прокинуть их аксес портами до соответствующих ПК
5. Настроить отдельный VRF для маршрутизации и привязать к L3 VNI
6. Настроить Anycast gateway
7. Проверить работоспособность L3 Vxlan

### 1. L3 схема сети
![alt text](image.png)

### 2. Выделить IP адрес для ПК из другой сети
Данный ПК подключен к Leaf2
```console
PC3> ip 192.168.2.1 255.255.255.0 192.168.2.254
```
### 3. Настройка VTEP на Leaf 2
```console
Leaf2(config)#service routing protocols model multi-agent
Leaf2#reload 
```
```console
Leaf2(config)#int loopback 3
Leaf2(config-if-Lo3)#description Vxlan
Leaf2(config-if-Lo3)#ip address 10.1.4.3 255.255.255.255
Leaf2(config-if-Lo3)#exit
Leaf2(config)#interface Vxlan1
Leaf2(config-if-Vx1)#vxlan source-interface Loopback3
```
также донастроим EVPN так как в прошлой работе Leaf2 не использовался
```console
Leaf2(config)#router bgp 65532
Leaf2(config-router-bgp)#neighbor 10.1.1.1 send-community extended
Leaf2(config-router-bgp)#neighbor 10.1.2.1 send-community extended
Leaf2(config-router-bgp)#address-family evpn 
Leaf2(config-router-bgp-af)#neighbor 10.1.1.1 activate
Leaf2(config-router-bgp-af)#neighbor 10.1.2.1 activate
Leaf2(config-router-bgp-af)#address-family ipv4
Leaf2(config-router-bgp-af)#network 10.1.4.3/32

Spine1(config)#router bgp 65534
Spine1(config-router-bgp)#neighbor 10.1.4.1 send-community extended
Spine1(config-router-bgp)#address-family evpn
Spine1(config-router-bgp-af)#neighbor 10.1.4.1 activate 

Spine2(config)#router bgp 65534
Spine2(config-router-bgp)#neighbor 10.1.4.1 send-community extended
Spine2(config-router-bgp)#address-family evpn
Spine2(config-router-bgp-af)#neighbor 10.1.4.1 activate 
```
### 4. Настроить Vlan 200 на Leaf 2 и прокинуть их аксес портами до соответствующих ПК.
```console
Leaf2(config)#vlan 200
Leaf2(config-vlan-200)#name PC_VRF
Leaf2(config-vlan-200)#exit
Leaf2(config)#int ethernet 8
Leaf2(config-if-Et8)#description ==PC3==
Leaf2(config-if-Et8)#sw mode access 
Leaf2(config-if-Et8)#sw acc v 200
```

### 5. Настроить отдельный VRF для маршрутизации и привязать к L3 VNI
Настроим VRF GLOBAL. Примапим в интерфейсе Vxlan к VNI и добавим в BGP.

```console
Leaf1(config)#vrf instance GLOBAL
Leaf1(config-vrf-GLOBAL)#ip routing vrf GLOBAL
Leaf1(config)#int vxlan 1
Leaf1(config-if-Vx1)#vxlan vrf GLOBAL vni 100001
Leaf1(config)#router bgp 65531
Leaf1(config-router-bgp)#vrf GLOBAL
Leaf1(config-router-bgp-vrf-GLOBAL)#rd 65531:100001
Leaf1(config-router-bgp-vrf-GLOBAL)#route-target import evpn 1:100001
Leaf1(config-router-bgp-vrf-GLOBAL)#route-target export evpn 1:100001      
Leaf1(config-router-bgp-vrf-GLOBAL)#redistribute connected 

Leaf2(config)#vrf instance GLOBAL
Leaf2(config-vrf-GLOBAL)#ip routing vrf GLOBAL
Leaf2(config)#int vxlan 1
Leaf2(config-if-Vx1)#vxlan vrf GLOBAL vni 100001
Leaf2(config-if-Vx1)#router BGP 65532
Leaf2(config-router-bgp)#vrf GLOBAL
Leaf2(config-router-bgp-vrf-GLOBAL)#rd 65532:100001
Leaf2(config-router-bgp-vrf-GLOBAL)#route-target import evpn 1:100001
Leaf2(config-router-bgp-vrf-GLOBAL)#route-target export evpn 1:100001      
Leaf2(config-router-bgp-vrf-GLOBAL)#redistribute connected

Leaf3(config)#vrf instance GLOBAL
Leaf3(config-vrf-GLOBAL)#ip routing vrf GLOBAL
Leaf3(config)#int vxlan 1
Leaf3(config-if-Vx1)#vxlan vrf GLOBAL vni 100001
Leaf3(config)#router bgp 65533           
Leaf3(config-router-bgp)#vrf GLOBAL
Leaf3(config-router-bgp-vrf-GLOBAL)#rd 65533:100001
Leaf3(config-router-bgp-vrf-GLOBAL)#route-target import evpn 1:100001
Leaf3(config-router-bgp-vrf-GLOBAL)#route-target export evpn 1:100001      
Leaf3(config-router-bgp-vrf-GLOBAL)#redistribute connected 
```
### 6. Настроить Anycast gateway
Leaf1(config)#int vlan 100
Leaf1(config-if-Vl100)#description ==PC==
Leaf1(config-if-Vl100)#vrf GLOBAL
Leaf1(config-if-Vl100)#ip address virtual 192.168.1.254/24

Leaf2(config)#int vlan 200
Leaf2(config-if-Vl200)#description ==PC_VRF==
Leaf2(config-if-Vl200)#vrf GLOBAL
Leaf2(config-if-Vl200)#ip address virtual 192.168.2.254/24

int Leaf3(config)#int vlan 100
Leaf3(config-if-Vl100)#description ==PC==
Leaf3(config-if-Vl100)#vrf GLOBAL
Leaf3(config-if-Vl100)#ip address virtual 192.168.1.254/24

### 7. Проверить работоспособность L3 Vxlan
Видим что Leaf2 имеет 2 VTEP соседа
```console
Leaf2#sh vxlan vtep 
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.1.3.3       unicast       
10.1.5.3       unicast       

Total number of remote VTEPS:  2
```
Проверим наличие 5-го типа маршрутов

```console
Leaf2#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.4.1, local AS number 65532
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65531:100001 ip-prefix 192.168.1.0/24
                                 10.1.3.3              -       100     0       65534 65531 i
 *        RD: 65531:100001 ip-prefix 192.168.1.0/24
                                 10.1.3.3              -       100     0       65534 65531 i
 * >      RD: 65533:100001 ip-prefix 192.168.1.0/24
                                 10.1.5.3              -       100     0       65534 65533 i
 *        RD: 65533:100001 ip-prefix 192.168.1.0/24
                                 10.1.5.3              -       100     0       65534 65533 i
 * >      RD: 65532:100001 ip-prefix 192.168.2.0/24
                                 -                     -       -       0       i
```
Проверим таблицу маршрутизации в VRF

```console
Leaf2#show ip route vrf GLOBAL

VRF: GLOBAL
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B E      192.168.1.1/32 [200/0] via VTEP 10.1.3.3 VNI 100001 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      192.168.1.2/32 [200/0] via VTEP 10.1.5.3 VNI 100001 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 B E      192.168.1.0/24 [200/0] via VTEP 10.1.5.3 VNI 100001 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        192.168.2.0/24 is directly connected, Vlan200
```
Проверим пинги всех ПК с ПК1 (за Leaf1) и с ПК2 (за Leaf2)

```console
PC1> ping 192.168.1.2

84 bytes from 192.168.1.2 icmp_seq=1 ttl=64 time=66.726 ms
84 bytes from 192.168.1.2 icmp_seq=2 ttl=64 time=48.770 ms
84 bytes from 192.168.1.2 icmp_seq=3 ttl=64 time=61.580 ms
84 bytes from 192.168.1.2 icmp_seq=4 ttl=64 time=35.757 ms
84 bytes from 192.168.1.2 icmp_seq=5 ttl=64 time=142.498 ms

PC1> ping 192.168.2.1

84 bytes from 192.168.2.1 icmp_seq=1 ttl=62 time=122.174 ms
84 bytes from 192.168.2.1 icmp_seq=2 ttl=62 time=109.108 ms
84 bytes from 192.168.2.1 icmp_seq=3 ttl=62 time=52.948 ms
84 bytes from 192.168.2.1 icmp_seq=4 ttl=62 time=47.426 ms
84 bytes from 192.168.2.1 icmp_seq=5 ttl=62 time=55.605 ms
```
```console
PC3> ping 192.168.1.1

84 bytes from 192.168.1.1 icmp_seq=1 ttl=62 time=50.471 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=62 time=56.940 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=62 time=79.049 ms
84 bytes from 192.168.1.1 icmp_seq=4 ttl=62 time=52.127 ms
84 bytes from 192.168.1.1 icmp_seq=5 ttl=62 time=40.017 ms

PC3> ping 192.168.1.2

84 bytes from 192.168.1.2 icmp_seq=1 ttl=62 time=229.605 ms
84 bytes from 192.168.1.2 icmp_seq=2 ttl=62 time=64.940 ms
84 bytes from 192.168.1.2 icmp_seq=3 ttl=62 time=47.396 ms
84 bytes from 192.168.1.2 icmp_seq=4 ttl=62 time=64.455 ms
84 bytes from 192.168.1.2 icmp_seq=5 ttl=62 time=41.573 ms
```