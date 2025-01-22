## Дз8 Оптимизация таблиц маршрутизации
### План работ:
1. Добавить L3 схему сети
2. Добавить таблицу с IP простаранством
3. Настроить дополнительный VRF и L3 VNI
4. Добавить клиента в новом VRF
5. Поднять порты между фабрикой и новым маршрутизатором
6. Поднять eBGP между маршрутизаторами и фабрикой
7. Настроить Route liking в VRF lite
8. Проверить внешние маршруты type 5


### 3. Настроить дополнительный VRF и L3 VNI
В качестве L3 VNI возьмем идентификатор 100002  
Новый VRF назовем CUSTOMER2

```console
Leaf1#conf t
Leaf1(config)#vrf instance CUSTOMER2
Leaf1(config-vrf-CUSTOMER2)#exit
Leaf1(config)#ip routing vrf CUSTOMER2 
Leaf1(config)#interface Vxlan1
Leaf1(config-if-Vx1)#vxlan vrf CUSTOMER2 vni 100002
Leaf1(config-if-Vx1)#router bgp 65531
Leaf1(config-router-bgp)#vrf CUSTOMER2 
Leaf1(config-router-bgp-vrf-CUSTOMER2)#rd 65531:100002
Leaf1(config-router-bgp-vrf-CUSTOMER2)#route-target import evpn 2:100002
Leaf1(config-router-bgp-vrf-CUSTOMER2)#route-target export evpn 2:100002      
Leaf1(config-router-bgp-vrf-CUSTOMER2)#redistribute connected 

Leaf1-2#conf t
Leaf1-2(config)#vrf instance CUSTOMER2
Leaf1-2(config-vrf-CUSTOMER2)#exit
Leaf1-2(config)#ip routing vrf CUSTOMER2 
Leaf1-2(config)#interface Vxlan1
Leaf1-2(config-if-Vx1)#vxlan vrf CUSTOMER2 vni 100002
Leaf1-2(config-if-Vx1)#router bgp 65528
Leaf1-2(config-router-bgp)#vrf CUSTOMER2 
Leaf1-2(config-router-bgp-vrf-CUSTOMER2)#rd 65528:100002
Leaf1-2(config-router-bgp-vrf-CUSTOMER2)#route-target import evpn 2:100002
Leaf1-2(config-router-bgp-vrf-CUSTOMER2)#route-target export evpn 2:100002       
Leaf1-2(config-router-bgp-vrf-CUSTOMER2)#redistribute connected 

Leaf3-2(config)#vrf instance CUSTOMER2
Leaf3-2(config-vrf-CUSTOMER2)#exit
Leaf3-2(config)#ip routing vrf CUSTOMER2 
Leaf3-2(config)#interface Vxlan1
Leaf3-2(config-if-Vx1)#vxlan vrf CUSTOMER2 vni 100002
Leaf3-2(config-if-Vx1)#router bgp 65529
Leaf3-2(config-router-bgp)#vrf CUSTOMER2 
Leaf3-2(config-router-bgp-vrf-CUSTOMER2)#rd 65529:100002
Leaf3-2(config-router-bgp-vrf-CUSTOMER2)#route-target import evpn 2:100002
Leaf3-2(config-router-bgp-vrf-CUSTOMER2)#route-target export evpn 2:100002       
Leaf3-2(config-router-bgp-vrf-CUSTOMER2)#redistribute connected 

Leaf3#conf t
Leaf3(config)#vrf instance CUSTOMER2
Leaf3(config-vrf-CUSTOMER2)#exit
Leaf3(config)#ip routing vrf CUSTOMER2 
Leaf3(config)#interface Vxlan1
Leaf3(config-if-Vx1)#vxlan vrf CUSTOMER2 vni 100002
Leaf3(config-if-Vx1)#router bgp 65533
Leaf3(config-router-bgp)#vrf CUSTOMER2 
Leaf3(config-router-bgp-vrf-CUSTOMER2)#rd 65533:100002
Leaf3(config-router-bgp-vrf-CUSTOMER2)#route-target import evpn 2:100002
Leaf3(config-router-bgp-vrf-CUSTOMER2)#route-target export evpn 2:100002      
Leaf3(config-router-bgp-vrf-CUSTOMER2)#redistribute connected 

Leaf2#conf t
Leaf2(config)#vrf instance CUSTOMER2
Leaf2(config-vrf-CUSTOMER2)#exit
Leaf2(config)#ip routing vrf CUSTOMER2 
Leaf2(config)#interface Vxlan1
Leaf2(config-if-Vx1)#vxlan vrf CUSTOMER2 vni 100002
Leaf2(config-if-Vx1)#router bgp 65532
Leaf2(config-router-bgp)#vrf CUSTOMER2 
Leaf2(config-router-bgp-vrf-CUSTOMER2)#rd 65532:100002
Leaf2(config-router-bgp-vrf-CUSTOMER2)#route-target import evpn 2:100002
Leaf2(config-router-bgp-vrf-CUSTOMER2)#route-target export evpn 2:100002      
Leaf2(config-router-bgp-vrf-CUSTOMER2)#redistribute connected 
```

### 4. Добавить клиента в новом VRF

Создадим в новом VRF Vlan 300 и привяжем его к VNI 100300

```console
Leaf1#conf t
Leaf1(config)#vlan 300
Leaf1(config-vlan-300)#name CUSTOMER2
Leaf1(config-vlan-300)#exit
Leaf1(config)#interface Vxlan1
Leaf1(config-if-Vx1)#vxlan vlan 300 vni 100300
Leaf1(config-if-Vx1)#router bgp 65531
Leaf1(config-router-bgp)#vlan 300
Leaf1(config-macvrf-300)#rd 65531:100300
Leaf1(config-macvrf-300)#route-target both 65531:100300
Leaf1(config-macvrf-300)#redistribute learned

Leaf1-2(config)#vlan 300
Leaf1-2(config-vlan-300)#name CUSTOMER2
Leaf1-2(config-vlan-300)#exit
Leaf1-2(config)#interface Vxlan1
Leaf1-2(config-if-Vx1)#vxlan vlan 300 vni 100300
Leaf1-2(config-if-Vx1)#router bgp 65528
Leaf1-2(config-router-bgp)#vlan 300
Leaf1-2(config-macvrf-300)#rd 65528:100300
Leaf1-2(config-macvrf-300)#route-target both 65531:100300
Leaf1-2(config-macvrf-300)#redistribute learned

Leaf3-2(config)#vlan 300
Leaf3-2(config-vlan-300)#name CUSTOMER2
Leaf3-2(config-vlan-300)#exit
Leaf3-2(config)#interface Vxlan1
Leaf3-2(config-if-Vx1)#vxlan vlan 300 vni 100300
Leaf3-2(config-if-Vx1)#router bgp 65529
Leaf3-2(config-router-bgp)#vlan 300
Leaf3-2(config-macvrf-300)#rd 65529:100300
Leaf3-2(config-macvrf-300)#route-target both 65531:100300
Leaf3-2(config-macvrf-300)#redistribute learned

Leaf3(config)#vlan 300
Leaf3(config-vlan-300)#name CUSTOMER2
Leaf3(config-vlan-300)#exit
Leaf3(config)#interface Vxlan1
Leaf3(config-if-Vx1)#vxlan vlan 300 vni 100300
Leaf3(config-if-Vx1)#router bgp 65533
Leaf3(config-router-bgp)#vlan 300
Leaf3(config-macvrf-300)#rd 65533:100300
Leaf3(config-macvrf-300)#route-target both 65531:100300
Leaf3(config-macvrf-300)#redistribute learned

Leaf2(config)#vlan 300
Leaf2(config-vlan-300)#name CUSTOMER2
Leaf2(config-vlan-300)#exit
Leaf2(config)#interface Vxlan1
Leaf2(config-if-Vx1)#vxlan vlan 300 vni 100300
Leaf2(config-if-Vx1)#router bgp 65532
Leaf2(config-router-bgp)#vlan 300
Leaf2(config-macvrf-300)#rd 65532:100300
Leaf2(config-macvrf-300)#route-target both 65531:100300
Leaf2(config-macvrf-300)#redistribute learned
```

Настроим новые пк и аксес порты для них

PC 4 подключен к Leaf2

```console
PC4> ip 192.168.3.1 255.255.255.0 192.168.3.254
Checking for duplicate address...
PC4 : 192.168.3.1 255.255.255.0 gateway 192.168.3.254
```

PC 5 подключен к кастомер свичу 2

```console
VPCS> set pcname PC5
PC5> ip 192.168.3.2 255.255.255.0 192.168.3.254
```

```console
Leaf2#conf t
Leaf2(config)#int ethernet 9
Leaf2(config-if-Et9)#description ==PC4==
Leaf2(config-if-Et9)#sw
Leaf2(config-if-Et9)#sw mode acc
Leaf2(config-if-Et9)#sw acc v 300

Leaf3(config)#interface Port-Channel1
Leaf3(config-if-Po1)#sw trunk allowed vlan add 300

Leaf3-2(config)#interface Port-Channel1
Leaf3-2(config-if-Po1)#sw trunk allowed vlan add 300

CUSTOMER-2(config)#vlan 300
CUSTOMER-2(config-vlan-300)#name CUSTOMER2
CUSTOMER-2(config-vlan-300)#exit
CUSTOMER-2(config)#int eth3
CUSTOMER-2(config-if-Et3)#description ==PC5==
CUSTOMER-2(config-if-Et3)#sw
CUSTOMER-2(config-if-Et3)#sw mode acc
CUSTOMER-2(config-if-Et3)#sw acc v 300
```

Настроим anycast gateway

```console
Leaf3(config)#interface Vlan300
Leaf3(config-if-Vl300)#vrf CUSTOMER2 
Leaf3(config-if-Vl300)#ip address virtual 192.168.3.254/24
Leaf3(config-if-Vl300)#no shutdown 

Leaf3-2(config)#int vlan 300
Leaf3-2(config-if-Vl300)#no shutdown 
Leaf3-2(config-if-Vl300)#vrf CUSTOMER2 
Leaf3-2(config-if-Vl300)#ip address virtual 192.168.3.254/24

Leaf2#conf t
Leaf2(config)#int vlan 300
Leaf2(config-if-Vl300)#vrf CUSTOMER2 
Leaf2(config-if-Vl300)#ip address 192.168.3.254/24
Leaf2(config-if-Vl300)#no shut
```

Проверим что внутри нового VRF пк пингуют друг друга

```console
PC5> ping 192.168.3.1

84 bytes from 192.168.3.1 icmp_seq=1 ttl=64 time=289.569 ms
84 bytes from 192.168.3.1 icmp_seq=2 ttl=64 time=246.913 ms
84 bytes from 192.168.3.1 icmp_seq=3 ttl=64 time=153.213 ms
84 bytes from 192.168.3.1 icmp_seq=4 ttl=64 time=143.164 ms
84 bytes from 192.168.3.1 icmp_seq=5 ttl=64 time=106.863 ms
```

### 5. Поднять портченнелы между фабрикой и новым маршрутизатором

создадим vlan и svi для роутинга между vrf

```console
Leaf3-2(config)#vlan 400
Leaf3-2(config-vlan-400)#int vlan 400
Leaf3-2(config-if-Vl400)#no shut
Leaf3-2(config-if-Vl400)#vrf GLOBAL
Leaf3-2(config-if-Vl400)#ip address 10.10.10.2/29
Leaf3-2(config-if-Vl400)#vlan 500
Leaf3-2(config-if-Vl500)#vrf CUSTOMER2 
Leaf3-2(config-vlan-500)#no shut
Leaf3-2(config-vlan-500)#int vlan 500
Leaf3-2(config-if-Vl500)#ip address 172.16.1.2/29

Leaf3(config)#vlan 400
Leaf3(config-vlan-400)#int vlan 400
Leaf3(config-if-Vl400)#vrf GLOBAL 
nLeaf3(config-if-Vl400)#no shut
Leaf3(config-if-Vl400)#ip address 10.10.10.1/29
Leaf3(config-if-Vl400)#vlan 500
Leaf3(config-vlan-500)#int vlan 500
Leaf3(config-if-Vl500)#vrf CUSTOMER2 
Leaf3(config-if-Vl500)#no shut
Leaf3(config-if-Vl500)#ip address 172.16.1.1/29
```

СНастроим порты между leaf 3, 3-2 и R01

```console

Leaf3-2(config)#int ethernet 4
Leaf3-2(config-if-Po10)#sw
Leaf3-2(config-if-Po10)#sw mode trunk 
Leaf3-2(config-if-Po10)#sw trunk allowed vlan 400,500

Leaf3(config)#int ethernet 4
Leaf3(config-if-Po10)#sw
Leaf3(config-if-Po10)#sw mode trunk 
Leaf3(config-if-Po10)#sw trunk allowed vlan 400,500

R01(config)#vlan 400
R01(config-vlan-400)#int vlan 400
R01(config-if-Vl400)#ip address 10.10.10.3/29
R01(config-if-Vl400)#no shut
R01(config-if-Vl400)#vlan 500
R01(config-vlan-500)#int vlan 500
R01(config-if-Vl500)#no shut
R01(config-if-Vl500)#ip address 172.16.1.3/29
R01(config-if-Vl500)#int eth1
R01(config-if-Et1)#sw
R01(config-if-Et1)#sw mode trunk 
R01(config-if-Et1)#int eth2
R01(config-if-Et2)#sw
R01(config-if-Et2)#sw mode trunk
```
### 6. Поднять eBGP между маршрутизаторами и фабрикой

```console
R01(config)#router bgp 65527
R01(config-router-bgp)#neighbor 10.10.10.1 remote-as 65533
R01(config-router-bgp)#neighbor 10.10.10.2 remote-as 65529
R01(config-router-bgp)#neighbor 172.16.1.1 remote-as 65533
R01(config-router-bgp)#neighbor 172.16.1.2 remote-as 65529
R01(config-router-bgp)#address-family ipv4 

Leaf3-2(config)#router bgp 65529
Leaf3-2(config-router-bgp)#vrf GLOBAL
Leaf3-2(config-router-bgp-vrf-GLOBAL)#neighbor 10.10.10.3 remote-as 65527
Leaf3-2(config-router-bgp-vrf-GLOBAL)#address-family ipv4 
Leaf3-2(config-router-bgp-vrf-GLOBAL-af)#exit
Leaf3-2(config-router-bgp-vrf-GLOBAL)#exit
Leaf3-2(config-router-bgp)#vrf CUSTOMER2
Leaf3-2(config-router-bgp-vrf-CUSTOMER2)#neighbor 172.16.1.3 remote-as 65527
Leaf3-2(config-router-bgp-vrf-CUSTOMER2)#address-family ipv4
Leaf3-2(config-router-bgp-vrf-CUSTOMER2-af)#exit

Leaf3(config)#router bgp 65533
Leaf3(config-router-bgp)#vrf GLOBAL
Leaf3(config-router-bgp-vrf-GLOBAL)#neighbor 10.10.10.3 remote-as 65527
Leaf3(config-router-bgp-vrf-GLOBAL)#address-family ipv4
Leaf3(config-router-bgp-vrf-GLOBAL-af)#exit
Leaf3(config-router-bgp-vrf-GLOBAL)#exit
Leaf3(config-router-bgp)#vrf CUSTOMER2 
Leaf3(config-router-bgp-vrf-CUSTOMER2)#neighbor 172.16.1.3 remote-as 65527
Leaf3(config-router-bgp-vrf-CUSTOMER2)#address-family ipv4 

R01#sh ip bgp summary 
BGP summary information for VRF default
Router identifier 172.16.1.3, local AS number 65527
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.10.10.1       4  65533              7        11    0    0 00:01:36 Estab   2      2
  10.10.10.2       4  65529              9        10    0    0 00:02:55 Estab   2      2
  172.16.1.1       4  65533              7        11    0    0 00:01:36 Estab   1      1
  172.16.1.2       4  65529              8        11    0    0 00:02:54 Estab   1      1
```
### 7. Настроить Route liking в VRF lite

```console
Leaf3-2(config)#ip prefix-list PERMITNETS seq 10 permit 192.168.0.0/16 eq 24
Leaf3-2(config)#route-map VRF1_Filter_Out permit 10 
Leaf3-2(config-route-map-VRF1_Filter_Out)#match ip address prefix-list PERMITNETS
Leaf3-2(config-route-map-VRF1_Filter_Out)#route-map VRF1_Filter_Out deny 100
Leaf3-2(config-route-map-VRF1_Filter_Out)#router bgp 65529
Leaf3-2(config-router-bgp)#vrf CUSTOMER2
Leaf3-2(config-router-bgp-vrf-CUSTOMER2)#address-family ipv4
Leaf3-2(config-router-bgp-vrf-CUSTOMER2-af)neighbor 172.16.1.3 route-map VRF1_F 
Leaf3-2(config-router-bgp-vrf-CUSTOMER2-af)#vrf GLOBAL
Leaf3-2(config-router-bgp-vrf-GLOBAL)#address-family ipv4
Leaf3-2(config-router-bgp-vrf-GLOBAL-af)#aneighbor 10.10.10.3 route-map VRF1_Filtilter_Out

Leaf3(config)#ip prefix-list PERMITNETS seq 10 permit 192.168.0.0/16 eq 24
Leaf3(config)#route-map VRF1_Filter_Out permit 10 
Leaf3(config-route-map-VRF1_Filter_Out)#match ip address prefix-list PERMITNETS
Leaf3(config-route-map-VRF1_Filter_Out)#route-map VRF1_Filter_Out deny 100
Leaf3(config-route-map-VRF1_Filter_Out)#router bgp 65533
Leaf3(config-router-bgp)#vrf CUSTOMER2
Leaf3(config-router-bgp-vrf-CUSTOMER2)#address-family ipv4
Leaf3(config-router-bgp-vrf-CUSTOMER2-af)neighbor 172.16.1.3 route-map VRF1_F 
Leaf3(config-router-bgp-vrf-CUSTOMER2-af)#vrf GLOBAL
Leaf3(config-router-bgp-vrf-GLOBAL)#address-family ipv4
Leaf3(config-router-bgp-vrf-GLOBAL-af)#aneighbor 10.10.10.3 route-map VRF1_Filtilter_Out
```
Проверим что пошли пинги между VRF

```console
PC5> ping 192.168.1.1 -t

192.168.1.1 icmp_seq=1 timeout
84 bytes from 192.168.1.1 icmp_seq=2 ttl=60 time=242.768 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=60 time=222.429 ms
84 bytes from 192.168.1.1 icmp_seq=4 ttl=60 time=254.118 ms
84 bytes from 192.168.1.1 icmp_seq=5 ttl=60 time=280.403 ms
84 bytes from 192.168.1.1 icmp_seq=6 ttl=60 time=158.379 ms
84 bytes from 192.168.1.1 icmp_seq=7 ttl=60 time=156.332 ms
84 bytes from 192.168.1.1 icmp_seq=8 ttl=60 time=312.036 ms
84 bytes from 192.168.1.1 icmp_seq=9 ttl=60 time=375.791 ms
```

Посмотрим на 5ый тип маршрутов

```console
Leaf3#sh bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 10.1.5.1, local AS number 65533
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65533:100001 ip-prefix 10.10.10.0/29
                                 -                     -       -       0       i
 * >      RD: 65529:100002 ip-prefix 172.16.1.0/29
                                 10.1.7.3              -       100     0       65534 65529 i
 *        RD: 65529:100002 ip-prefix 172.16.1.0/29
                                 10.1.7.3              -       100     0       65534 65529 i
 * >      RD: 65533:100002 ip-prefix 172.16.1.0/29
                                 -                     -       -       0       i
 * >      RD: 65528:100001 ip-prefix 192.168.1.0/24
                                 10.1.6.3              -       100     0       65534 65528 i
 *        RD: 65528:100001 ip-prefix 192.168.1.0/24
                                 10.1.6.3              -       100     0       65534 65528 i
 * >      RD: 65531:100001 ip-prefix 192.168.1.0/24
                                 10.1.3.3              -       100     0       65534 65531 i
 *        RD: 65531:100001 ip-prefix 192.168.1.0/24
                                 10.1.3.3              -       100     0       65534 65531 i
 * >      RD: 65533:100001 ip-prefix 192.168.1.0/24
                                 -                     -       -       0       i
 * >      RD: 65532:100001 ip-prefix 192.168.2.0/24
                                 10.1.4.3              -       100     0       65534 65532 i
 *        RD: 65532:100001 ip-prefix 192.168.2.0/24
                                 10.1.4.3              -       100     0       65534 65532 i
 * >      RD: 65529:100002 ip-prefix 192.168.3.0/24
                                 10.1.7.3              -       100     0       65534 65529 i
 *        RD: 65529:100002 ip-prefix 192.168.3.0/24
                                 10.1.7.3              -       100     0       65534 65529 i
 * >      RD: 65532:100002 ip-prefix 192.168.3.0/24
                                 10.1.4.3              -       100     0       65534 65532 i
 *        RD: 65532:100002 ip-prefix 192.168.3.0/24
                                 10.1.4.3              -       100     0       65534 65532 i
 * >      RD: 65533:100002 ip-prefix 192.168.3.0/24
                                 -                     -       -       0       i
```
```console
Leaf3#sh bgp evpn 
BGP routing table information for VRF default
Router identifier 10.1.5.1, local AS number 65533
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65529:100300 auto-discovery 0 0050:0000:0000:000c:0001
                                 10.1.7.3              -       100     0       65534 65529 i
 *  ec    RD: 65529:100300 auto-discovery 0 0050:0000:0000:000c:0001
                                 10.1.7.3              -       100     0       65534 65529 i
 * >Ec    RD: 65533:100100 auto-discovery 0 0050:0000:0000:000c:0001
                                 10.1.7.3              -       100     0       65534 65529 i
 *  ec    RD: 65533:100100 auto-discovery 0 0050:0000:0000:000c:0001
                                 10.1.7.3              -       100     0       65534 65529 i
 * >Ec    RD: 10.1.7.3:1 auto-discovery 0050:0000:0000:000c:0001
                                 10.1.7.3              -       100     0       65534 65529 i
 *  ec    RD: 10.1.7.3:1 auto-discovery 0050:0000:0000:000c:0001
                                 10.1.7.3              -       100     0       65534 65529 i
 * >Ec    RD: 65528:100100 auto-discovery 0 0050:0000:0000:000d:0001
                                 10.1.6.3              -       100     0       65534 65528 i
 *  ec    RD: 65528:100100 auto-discovery 0 0050:0000:0000:000d:0001
                                 10.1.6.3              -       100     0       65534 65528 i
 * >Ec    RD: 65531:100100 auto-discovery 0 0050:0000:0000:000d:0001
                                 10.1.3.3              -       100     0       65534 65531 i
 *  ec    RD: 65531:100100 auto-discovery 0 0050:0000:0000:000d:0001
                                 10.1.3.3              -       100     0       65534 65531 i
 * >Ec    RD: 10.1.3.3:1 auto-discovery 0050:0000:0000:000d:0001
                                 10.1.3.3              -       100     0       65534 65531 i
 *  ec    RD: 10.1.3.3:1 auto-discovery 0050:0000:0000:000d:0001
                                 10.1.3.3              -       100     0       65534 65531 i
 * >Ec    RD: 10.1.6.3:1 auto-discovery 0050:0000:0000:000d:0001
                                 10.1.6.3              -       100     0       65534 65528 i
 *  ec    RD: 10.1.6.3:1 auto-discovery 0050:0000:0000:000d:0001
                                 10.1.6.3              -       100     0       65534 65528 i
 * >Ec    RD: 65528:100100 mac-ip 0050.7966.6806 192.168.1.1
                                 10.1.6.3              -       100     0       65534 65528 i
 *  ec    RD: 65528:100100 mac-ip 0050.7966.6806 192.168.1.1
                                 10.1.6.3              -       100     0       65534 65528 i
 * >Ec    RD: 65531:100100 mac-ip 0050.7966.6806 192.168.1.1
                                 10.1.3.3              -       100     0       65534 65531 i
 *  ec    RD: 65531:100100 mac-ip 0050.7966.6806 192.168.1.1
                                 10.1.3.3              -       100     0       65534 65531 i
 * >Ec    RD: 65529:100300 mac-ip 0050.7966.6808
                                 10.1.7.3              -       100     0       65534 65529 i
 *  ec    RD: 65529:100300 mac-ip 0050.7966.6808
                                 10.1.7.3              -       100     0       65534 65529 i
 * >Ec    RD: 65529:100300 mac-ip 0050.7966.6808 192.168.3.2
                                 10.1.7.3              -       100     0       65534 65529 i
 *  ec    RD: 65529:100300 mac-ip 0050.7966.6808 192.168.3.2
                                 10.1.7.3              -       100     0       65534 65529 i
 * >Ec    RD: 65531:100100 imet 10.1.3.3
                                 10.1.3.3              -       100     0       65534 65531 i
 *  ec    RD: 65531:100100 imet 10.1.3.3
                                 10.1.3.3              -       100     0       65534 65531 i
 * >Ec    RD: 65531:100300 imet 10.1.3.3
                                 10.1.3.3              -       100     0       65534 65531 i
 *  ec    RD: 65531:100300 imet 10.1.3.3
                                 10.1.3.3              -       100     0       65534 65531 i
 * >Ec    RD: 65532:100300 imet 10.1.4.3
                                 10.1.4.3              -       100     0       65534 65532 i
 *  ec    RD: 65532:100300 imet 10.1.4.3
                                 10.1.4.3              -       100     0       65534 65532 i
 * >      RD: 65533:100100 imet 10.1.5.3
                                 -                     -       -       0       i
 * >      RD: 65533:100300 imet 10.1.5.3
                                 -                     -       -       0       i
 * >Ec    RD: 65528:100100 imet 10.1.6.3
                                 10.1.6.3              -       100     0       65534 65528 i
 *  ec    RD: 65528:100100 imet 10.1.6.3
                                 10.1.6.3              -       100     0       65534 65528 i
 * >Ec    RD: 65528:100300 imet 10.1.6.3
                                 10.1.6.3              -       100     0       65534 65528 i
 *  ec    RD: 65528:100300 imet 10.1.6.3
                                 10.1.6.3              -       100     0       65534 65528 i
 * >Ec    RD: 65529:100300 imet 10.1.7.3
                                 10.1.7.3              -       100     0       65534 65529 i
 *  ec    RD: 65529:100300 imet 10.1.7.3
                                 10.1.7.3              -       100     0       65534 65529 i
 * >Ec    RD: 65533:100100 imet 10.1.7.3
                                 10.1.7.3              -       100     0       65534 65529 i
 *  ec    RD: 65533:100100 imet 10.1.7.3
                                 10.1.7.3              -       100     0       65534 65529 i
 * >Ec    RD: 10.1.7.3:1 ethernet-segment 0050:0000:0000:000c:0001 10.1.7.3
                                 10.1.7.3              -       100     0       65534 65529 i
 *  ec    RD: 10.1.7.3:1 ethernet-segment 0050:0000:0000:000c:0001 10.1.7.3
                                 10.1.7.3              -       100     0       65534 65529 i
 * >Ec    RD: 10.1.3.3:1 ethernet-segment 0050:0000:0000:000d:0001 10.1.3.3
                                 10.1.3.3              -       100     0       65534 65531 i
 *  ec    RD: 10.1.3.3:1 ethernet-segment 0050:0000:0000:000d:0001 10.1.3.3
                                 10.1.3.3              -       100     0       65534 65531 i
 * >Ec    RD: 10.1.6.3:1 ethernet-segment 0050:0000:0000:000d:0001 10.1.6.3
                                 10.1.6.3              -       100     0       65534 65528 i
 *  ec    RD: 10.1.6.3:1 ethernet-segment 0050:0000:0000:000d:0001 10.1.6.3
                                 10.1.6.3              -       100     0       65534 65528 i
 * >      RD: 65533:100001 ip-prefix 10.10.10.0/29
                                 -                     -       -       0       i
 * >      RD: 65529:100002 ip-prefix 172.16.1.0/29
                                 10.1.7.3              -       100     0       65534 65529 i
 *        RD: 65529:100002 ip-prefix 172.16.1.0/29
                                 10.1.7.3              -       100     0       65534 65529 i
 * >      RD: 65533:100002 ip-prefix 172.16.1.0/29
                                 -                     -       -       0       i
 * >      RD: 65528:100001 ip-prefix 192.168.1.0/24
                                 10.1.6.3              -       100     0       65534 65528 i
 *        RD: 65528:100001 ip-prefix 192.168.1.0/24
                                 10.1.6.3              -       100     0       65534 65528 i
 * >      RD: 65531:100001 ip-prefix 192.168.1.0/24
                                 10.1.3.3              -       100     0       65534 65531 i
 *        RD: 65531:100001 ip-prefix 192.168.1.0/24
                                 10.1.3.3              -       100     0       65534 65531 i
 * >      RD: 65533:100001 ip-prefix 192.168.1.0/24
                                 -                     -       -       0       i
 * >      RD: 65532:100001 ip-prefix 192.168.2.0/24
                                 10.1.4.3              -       100     0       65534 65532 i
 *        RD: 65532:100001 ip-prefix 192.168.2.0/24
                                 10.1.4.3              -       100     0       65534 65532 i
 * >      RD: 65529:100002 ip-prefix 192.168.3.0/24
                                 10.1.7.3              -       100     0       65534 65529 i
 *        RD: 65529:100002 ip-prefix 192.168.3.0/24
                                 10.1.7.3              -       100     0       65534 65529 i
 * >      RD: 65532:100002 ip-prefix 192.168.3.0/24
                                 10.1.4.3              -       100     0       65534 65532 i
 *        RD: 65532:100002 ip-prefix 192.168.3.0/24
                                 10.1.4.3              -       100     0       65534 65532 i
 * >      RD: 65533:100002 ip-prefix 192.168.3.0/24
                                 -                     -       -       0       i
```