## Дз4 Построение Underlay сети(eBGP)
### План работ:
1. Добавить L3 схему сети
2. Добавить таблицу с ip пространством
3. Выпиливание протокола IS-IS
4. Исключение лупбеков на лиф коммутаторах из OSPF
5. Включить на всех лиф устройствах BGP
6. Настроить на всех лиф устройствах RID
7. Поднять соседство между всеми Leaf коммутаторами
8. Настроить на Spine коммутаторах BGP
9. Настроить BFD
10. Проверить доступность всех лупбеков в BGP

### 1. L3 схема сети
![alt text](../ДЗ1_Основы_Проектирования_Сети/image.png)

### 2. Таблица с ip пространством
| network ipv4 | Device/port|    Description    |
|--------------|:----------:| -----------------:|
| 10.1.1.1/32  | Spine1/lo1 |     Loopback1     |
| 10.1.1.2/32  | Spine1/lo2 |     Loopback2     |
| 10.1.2.1/32  | Spine2/lo1 |     Loopback1     |
| 10.1.2.2/32  | Spine2/lo2 |     Loopback2     |
| 10.1.3.1/32  |  Leaf1/lo1 |     Loopback1     |
| 10.1.3.2/32  |  Leaf1/lo2 |     Loopback2     |
| 10.1.4.1/32  |  Leaf2/lo1 |     Loopback1     |
| 10.1.4.2/32  |  Leaf2/lo2 |     Loopback2     |
| 10.1.5.1/32  |  Leaf3/lo1 |     Loopback1     |
| 10.1.5.2/32  |  Leaf3/lo2 |     Loopback2     |
| 10.1.1.5/30  | Spine1/Eth1| P2P Link To Leaf1 |
| 10.1.1.9/30  | Spine1/Eth2| P2P Link To Leaf2 |
| 10.1.1.13/30 | Spine1/Eth3| P2P Link To Leaf3 |
| 10.1.1.6/30  | Leaf1/Eth1 | P2P Link To Spine1|
| 10.1.2.6/30  | Leaf1/Eth2 | P2P Link To Spine2|
| 10.1.2.5/30  | Spine2/Eth1| P2P Link To Leaf1 |
| 10.1.2.9/30  | Spine2/Eth2| P2P Link To Leaf2 |
| 10.1.2.13/30 | Spine2/Eth2| P2P Link To Leaf3 |
| 10.1.1.10/30 | Leaf2/Eth1 | P2P Link To Spine1|
| 10.1.2.10/30 | Leaf2/Eth2 | P2P Link To Spine2|
| 10.1.1.14/30 | Leaf3/Eth1 | P2P Link To Spine1|
| 10.1.2.14/30 | Leaf3/Eth2 | P2P Link To Spine2|

### 3. Выпиливание протокола IS-IS
Так как протокол IS-IS в данное время не будет больше использоваться в работе, выпиливаем его со всех устройств

```console
Spine1(config)#interface Ethernet1 - 3
Spine1(config-if-Et1-3)#no isis authentication key
Spine1(config-if-Et1-3)#no isis authentication mode md5
Spine1(config-if-Et1-3)#no isis network point-to-point
Spine1(config-if-Et1-3)#no isis circuit-type level-1
Spine1(config-if-Et1-3)#no isis enable 99
Spine1(config-if-Et1-3)#exit
Spine1(config)#interface Loopback1 -2
Spine1(config-if-Lo1-2)#no isis circuit-type level-1
Spine1(config-if-Lo1-2)#no isis enable 99
Spine1(config)#no router isis 99

Spine2(config)#int Ethernet1 - 3
Spine2(config-if-Et1-3)#no isis authentication key
Spine2(config-if-Et1-3)#no isis authentication mode md5
Spine2(config-if-Et1-3)#no isis network point-to-point
Spine2(config-if-Et1-3)#no isis circuit-type level-1
Spine2(config-if-Et1-3)#no isis enable 99
Spine2(config-if-Et1-3)#exit
Spine2(config)#int loopback 1 - 2
Spine2(config-if-Lo1-2)#no isis circuit-type level-1
Spine2(config-if-Lo1-2)#no isis enable 99
Spine2(config)#no router isis 99

Leaf1(config)#interface Ethernet1 - 2
Leaf1(config-if-Et1-2)#no isis authentication key
Leaf1(config-if-Et1-2)#no isis authentication mode md5
Leaf1(config-if-Et1-2)#no isis network point-to-point
Leaf1(config-if-Et1-2)#no isis circuit-type level-1
Leaf1(config-if-Et1-2)#no isis enable 99
Leaf1(config-if-Et1-2)#int loopback 1 - 2
Leaf1(config-if-Lo1-2)#no isis circuit-type level-1
Leaf1(config-if-Lo1-2)#no isis enable 99
Leaf1(config-if-Lo1-2)#exit
Leaf1(config)#no router isis 99

Leaf2(config)#interface ethernet 1 - 2
Leaf2(config-if-Et1-2)#no isis authentication key
Leaf2(config-if-Et1-2)#no isis authentication mode md5
Leaf2(config-if-Et1-2)#no isis network point-to-point
Leaf2(config-if-Et1-2)#no isis circuit-type level-1
Leaf2(config-if-Et1-2)#no isis enable 99
Leaf2(config-if-Et1-2)#int loopback 1 - 2
Leaf2(config-if-Lo1-2)#no isis circuit-type level-1
Leaf2(config-if-Lo1-2)#no isis enable 99
Leaf2(config-if-Lo1-2)#exit
Leaf2(config)#no router isis 99

Leaf3(config)#int ethernet 1 - 2
\Leaf3(config-if-Et1-2)no isis authentication key
Leaf3(config-if-Et1-2)#no isis authentication mode md5
Leaf3(config-if-Et1-2)#no isis network point-to-point
Leaf3(config-if-Et1-2)#no isis circuit-type level-1
Leaf3(config-if-Et1-2)#no isis enable 99
Leaf3(config-if-Et1-2)#int loopback 1 - 2
Leaf3(config-if-Lo1-2)#no isis circuit-type level-1
Leaf3(config-if-Lo1-2)#no isis enable 99
Leaf3(config-if-Lo1-2)#exit
Leaf3(config)#no router isis 99
```

Проверяем что у нас все попрежнему доступно через OSPF

```console
Spine1#sh ip route

VRF: default
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

 C        10.1.1.1/32 is directly connected, Loopback1
 C        10.1.1.2/32 is directly connected, Loopback2
 C        10.1.1.4/30 is directly connected, Ethernet1
 C        10.1.1.8/30 is directly connected, Ethernet2
 C        10.1.1.12/30 is directly connected, Ethernet3
 O        10.1.2.1/32 [120/30] via 10.1.1.6, Ethernet1
                               via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 O        10.1.2.2/32 [120/30] via 10.1.1.6, Ethernet1
                               via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 O        10.1.2.4/30 [120/20] via 10.1.1.6, Ethernet1
 O        10.1.2.8/30 [120/20] via 10.1.1.10, Ethernet2
 O        10.1.2.12/30 [120/20] via 10.1.1.14, Ethernet3
 O        10.1.3.1/32 [120/20] via 10.1.1.6, Ethernet1
 O        10.1.3.2/32 [120/20] via 10.1.1.6, Ethernet1
 O        10.1.4.1/32 [120/20] via 10.1.1.10, Ethernet2
 O        10.1.4.2/32 [120/20] via 10.1.1.10, Ethernet2
 O        10.1.5.1/32 [120/20] via 10.1.1.14, Ethernet3
 O        10.1.5.2/32 [120/20] via 10.1.1.14, Ethernet3
```

### 4. Исключение лупбеков на лиф коммутаторах из OSPF
Исключаем лупбеки из OSPF на лиф коммутаторах (кроме будущего роутер id для BGP) так как в последующем настроим BGP для анонса лупбека лифов для проверки работы BGP. Все физические интерфейсы и лупбеки спайн коммутаторов попрежнему будут находиться в OSPF.

```console
Leaf1(config-if-Lo1)#int loopback 2
Leaf1(config-if-Lo2)#no ip ospf area 0.0.0.0

Leaf2(config-if-Lo1)#int loopback 2
Leaf2(config-if-Lo2)#no ip ospf area 0.0.0.0

Leaf3(config-if-Lo1)#interface Loopback2
Leaf3(config-if-Lo2)#no ip ospf area 0.0.0.0
```
Видим, что лупбеки пропали из таблици маршрутизации

```console

Leaf1#sh ip route

VRF: default
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

 O        10.1.1.1/32 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.1.2/32 [120/20] via 10.1.1.5, Ethernet1
 C        10.1.1.4/30 is directly connected, Ethernet1
 O        10.1.1.8/30 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.1.12/30 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.2.1/32 [120/20] via 10.1.2.5, Ethernet2
 O        10.1.2.2/32 [120/20] via 10.1.2.5, Ethernet2
 C        10.1.2.4/30 is directly connected, Ethernet2
 O        10.1.2.8/30 [120/20] via 10.1.2.5, Ethernet2
 O        10.1.2.12/30 [120/20] via 10.1.2.5, Ethernet2
 C        10.1.3.1/32 is directly connected, Loopback1
 C        10.1.3.2/32 is directly connected, Loopback2
 O        10.1.4.1/32 [120/30] via 10.1.1.5, Ethernet1
                               via 10.1.2.5, Ethernet2
 O        10.1.5.1/32 [120/30] via 10.1.1.5, Ethernet1
                               via 10.1.2.5, Ethernet2
```

Так как в качестве RID лиф коммутаторов выступает Lo1 мы всё равно будем их видеть в LSDB


```console
Leaf3#sh ip ospf database 

            OSPF Router with ID(10.1.5.1) (Instance ID 99) (VRF default)


                 Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum Link count
10.1.2.1        10.1.2.1        1453        0x80000ef7   0xabe5   8
10.1.4.1        10.1.4.1        200         0x80000a78   0x5e73   4
10.1.3.1        10.1.3.1        317         0x800009d8   0xd0b3   4
10.1.1.1        10.1.1.1        1477        0x80000db7   0xa735   8
10.1.5.1        10.1.5.1        157         0x80000940   0xa157   4
```

### 5. Включить на всех лиф устройствах BGP

На всех лифаг будут разные AS. (eBGP)

```console
Leaf1(config)#router bgp 65531

Leaf2(config)#router bgp 65532

Leaf3(config)#router bgp 65533
```

### 6. Настроить на всех лиф устройствах RID
В кач RID на лиф коммутаторах в процессе BGP будут выступать интерфейсы lo1

```console
Leaf1(config)#router bgp 65531
Leaf1(config-router-bgp)#router-id 10.1.3.1 

Leaf2(config)#router bgp 65532
Leaf2(config-router-bgp)#router-id 10.1.4.1

Leaf3(config)#router bgp 65533
Leaf3(config-router-bgp)#router-id 10.1.5.1
```

### 7. Поднять соседство между всеми Leaf коммутаторами

Подробно разберем соседства между leaf1 и leaf2:  
Сначала настроим соседей eBGP и активируем их

```console
Leaf1(config)#router bgp 65531
Leaf1(config-router-bgp)#neighbor 10.1.4.1 remote-as 65532
Leaf1(config-router-bgp)#address-family ipv4 
Leaf1(config-router-bgp-af)#neighbor 10.1.4.1 activate

Leaf2(config)#router bgp 65532
Leaf2(config-router-bgp)#neighbor 10.1.3.1 remote-as 65531
Leaf2(config-router-bgp)#address-family ipv4 
Leaf2(config-router-bgp-af)#neighbor 10.1.3.1 activate 
```

Видим что соседство не поднимается

```console
Leaf1#sh ip bgp summary 
BGP summary information for VRF default
Router identifier 10.1.3.1, local AS number 65531
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.4.1         4  65532              0         0    0    0    1d01h Active   
```

Дополнительно укажем update source - loopback 1 после чего соседство поднимится

```console
Leaf1(config)#router bgp 65531
Leaf1(config-router-bgp)#neighbor 10.1.4.1 update-source loopback 1

Leaf2(config)#router bgp 65532
Leaf2(config-router-bgp)#neighbor 10.1.3.1 update-source loopback 1

Leaf2#sh ip bgp summary 
BGP summary information for VRF default
Router identifier 10.1.4.1, local AS number 65532
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.3.1         4  65531              4         4    0    0 00:00:39 Estab   0      0
```

Проверим таблицу маршрутизации

```console
Leaf1#sh ip route

VRF: default
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

 O        10.1.1.1/32 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.1.2/32 [120/20] via 10.1.1.5, Ethernet1
 C        10.1.1.4/30 is directly connected, Ethernet1
 O        10.1.1.8/30 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.1.12/30 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.2.1/32 [120/20] via 10.1.2.5, Ethernet2
 O        10.1.2.2/32 [120/20] via 10.1.2.5, Ethernet2
 C        10.1.2.4/30 is directly connected, Ethernet2
 O        10.1.2.8/30 [120/20] via 10.1.2.5, Ethernet2
 O        10.1.2.12/30 [120/20] via 10.1.2.5, Ethernet2
 C        10.1.3.1/32 is directly connected, Loopback1
 C        10.1.3.2/32 is directly connected, Loopback2
 O        10.1.4.1/32 [120/30] via 10.1.1.5, Ethernet1
                               via 10.1.2.5, Ethernet2
 O        10.1.5.1/32 [120/30] via 10.1.1.5, Ethernet1
                               via 10.1.2.5, Ethernet2
```

Видим что по BGP ничего не прилетает. Добавим lo2 в кач анонсируемой сети и проверим таблицу маршрутизации

```console
Leaf1(config)#router bgp 65531
Leaf1(config-router-bgp)#address-family ipv4
Leaf1(config-router-bgp-af)#net 10.1.3.2 mask 255.255.255.255

Leaf2(config)#router bgp 65532
Leaf2(config-router-bgp)#address-family ipv4
Leaf2(config-router-bgp-af)#network 10.1.4.2 mask 255.255.255.255

Leaf1#sh ip route

VRF: default
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

 O        10.1.1.1/32 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.1.2/32 [120/20] via 10.1.1.5, Ethernet1
 C        10.1.1.4/30 is directly connected, Ethernet1
 O        10.1.1.8/30 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.1.12/30 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.2.1/32 [120/20] via 10.1.2.5, Ethernet2
 O        10.1.2.2/32 [120/20] via 10.1.2.5, Ethernet2
 C        10.1.2.4/30 is directly connected, Ethernet2
 O        10.1.2.8/30 [120/20] via 10.1.2.5, Ethernet2
 O        10.1.2.12/30 [120/20] via 10.1.2.5, Ethernet2
 C        10.1.3.1/32 is directly connected, Loopback1
 C        10.1.3.2/32 is directly connected, Loopback2
 O        10.1.4.1/32 [120/30] via 10.1.1.5, Ethernet1
                               via 10.1.2.5, Ethernet2
 B E      10.1.4.2/32 [200/0] via 10.1.1.5, Ethernet1
                              via 10.1.2.5, Ethernet2
 O        10.1.5.1/32 [120/30] via 10.1.1.5, Ethernet1
                               via 10.1.2.5, Ethernet2
```
Донастроим соседства между остальными лиф коммутаторами и анонсируем lo2 на leaf3 и выведем команды для проверки

```console
Leaf1(config)#router bgp 65531
Leaf1(config-router-bgp)#neighbor 10.1.5.1 remote-as 65533
Leaf1(config-router-bgp)#neighbor 10.1.5.1 update-source loopback 1
Leaf1(config-router-bgp)#neighbor 10.1.5.1 ebgp-multihop 
Leaf1(config-router-bgp)#address-family ipv4
Leaf1(config-router-bgp-af)#neighbor 10.1.5.1 activate 

Leaf2(config)#router bgp 65532
Leaf2(config-router-bgp)#neighbor 10.1.5.1 remote-as 65533
Leaf2(config-router-bgp)#neighbor 10.1.5.1 update-source loopback 1
Leaf2(config-router-bgp)#neighbor 10.1.5.1 ebgp-multihop 
Leaf2(config-router-bgp)#address-family ipv4
Leaf2(config-router-bgp-af)#neighbor 10.1.5.1 activate 

Leaf3(config)#router bgp 65533
Leaf3(config-router-bgp)#neighbor 10.1.3.1 remote-as 65531
Leaf3(config-router-bgp)#neighbor 10.1.3.1 update-source loopback 1
Leaf3(config-router-bgp)#neighbor 10.1.3.1 ebgp-multihop 
Leaf3(config-router-bgp)#address-family ipv4 
Leaf3(config-router-bgp-af)#neighbor 10.1.3.1 activate 
Leaf3(config-router-bgp-af)#network 10.1.5.2 mask 255.255.255.255
Leaf3(config-router-bgp-af)#exit
Leaf3(config-router-bgp)#neighbor 10.1.4.1 remote-as 65532
Leaf3(config-router-bgp)#neighbor 10.1.4.1 update-source loopback 1
Leaf3(config-router-bgp)#neighbor 10.1.4.1 ebgp-multihop 
Leaf3(config-router-bgp)#address-family ipv4
Leaf3(config-router-bgp-af)#neighbor 10.1.4.1 activate 

```
Видим что маршруты появились но не пингуется. трасерт умирает на спайнах потому что спайны не знают маршруты до сетей. настроим и на них BGP. достаточно будет настроить их в одной AS и без соседств друг с другом.

```console
Leaf1#sh ip route

VRF: default
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

 O        10.1.1.1/32 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.1.2/32 [120/20] via 10.1.1.5, Ethernet1
 C        10.1.1.4/30 is directly connected, Ethernet1
 O        10.1.1.8/30 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.1.12/30 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.2.1/32 [120/20] via 10.1.2.5, Ethernet2
 O        10.1.2.2/32 [120/20] via 10.1.2.5, Ethernet2
 C        10.1.2.4/30 is directly connected, Ethernet2
 O        10.1.2.8/30 [120/20] via 10.1.2.5, Ethernet2
 O        10.1.2.12/30 [120/20] via 10.1.2.5, Ethernet2
 C        10.1.3.1/32 is directly connected, Loopback1
 C        10.1.3.2/32 is directly connected, Loopback2
 O        10.1.4.1/32 [120/30] via 10.1.1.5, Ethernet1
                               via 10.1.2.5, Ethernet2
 B E      10.1.4.2/32 [200/0] via 10.1.1.5, Ethernet1
                              via 10.1.2.5, Ethernet2
 O        10.1.5.1/32 [120/30] via 10.1.1.5, Ethernet1
                               via 10.1.2.5, Ethernet2
 B E      10.1.5.2/32 [200/0] via 10.1.1.5, Ethernet1
                              via 10.1.2.5, Ethernet2

Leaf1#ping 10.1.4.2
PING 10.1.4.2 (10.1.4.2) 72(100) bytes of data.
From 10.1.1.5 icmp_seq=1 Destination Net Unreachable
tracer
--- 10.1.4.2 ping statistics ---
5 packets transmitted, 0 received, +1 errors, 100% packet loss, time 51ms
pipe 2
Leaf1#traceroute 10.1.4.2
traceroute to 10.1.4.2 (10.1.4.2), 30 hops max, 60 byte packets
 1  10.1.1.5 (10.1.1.5)  61.989 ms !N * *
```

8. Настроить на Spine коммутаторах BGP
Настроим соседей BGP через спайн коммутаторы

```console
Spine1(config)#router bgp 65534
Spine1(config-router-bgp)#router-id 10.1.1.1 
Spine1(config-router-bgp)#neighbor 10.1.3.1 remote-as 65531
Spine1(config-router-bgp)#neighbor 10.1.3.1 ebgp-multihop 
Spine1(config-router-bgp)#neighbor 10.1.3.1 update-source loopback 1
Spine1(config-router-bgp)#neighbor 10.1.4.1 remote-as 65532
Spine1(config-router-bgp)#neighbor 10.1.4.1 ebgp-multihop 
Spine1(config-router-bgp)#neighbor 10.1.4.1 update-source loopback 1 
Spine1(config-router-bgp)#neighbor 10.1.5.1 remote-as 65533
Spine1(config-router-bgp)#neighbor 10.1.5.1 ebgp-multihop 
Spine1(config-router-bgp)#neighbor 10.1.5.1 update-source loopback 1 
Spine1(config-router-bgp)#address-family ipv4
Spine1(config-router-bgp-af)#neighbor 10.1.3.1 activate 
Spine1(config-router-bgp-af)#neighbor 10.1.4.1 activate 
Spine1(config-router-bgp-af)#neighbor 10.1.5.1 activate 

Spine2(config)#router bgp 65534
Spine2(config-router-bgp)#router-id 10.1.2.1
Spine2(config-router-bgp)#neighbor 10.1.3.1 remote-as 65531
Spine2(config-router-bgp)#neighbor 10.1.3.1 ebgp-multihop 
Spine2(config-router-bgp)#neighbor 10.1.3.1 update-source loopback 1
Spine2(config-router-bgp)#neighbor 10.1.4.1 remote-as 65532
Spine2(config-router-bgp)#neighbor 10.1.4.1 ebgp-multihop 
Spine2(config-router-bgp)#neighbor 10.1.4.1 update-source loopback 1
Spine2(config-router-bgp)#neighbor 10.1.5.1 remote-as 65533
Spine2(config-router-bgp)#neighbor 10.1.5.1 ebgp-multihop 
Spine2(config-router-bgp)#neighbor 10.1.5.1 update-source loopback 1
Spine2(config-router-bgp)#address-family ipv4 
Spine2(config-router-bgp-af)#neighbor 10.1.3.1 activate 
Spine2(config-router-bgp-af)#neighbor 10.1.4.1 activate 
Spine2(config-router-bgp-af)#neighbor 10.1.5.1 activate 

Leaf1(config)#router bgp 65531
Leaf1(config-router-bgp)#neighbor 10.1.1.1 remote-as 65534
Leaf1(config-router-bgp)#neighbor 10.1.1.1 ebgp-multihop 
Leaf1(config-router-bgp)#neighbor 10.1.1.1 update-source loopback 1
Leaf1(config-router-bgp)#neighbor 10.1.2.1 remote-as 65534 
Leaf1(config-router-bgp)#neighbor 10.1.2.1 ebgp-multihop 
Leaf1(config-router-bgp)#neighbor 10.1.2.1 update-source loopback 1
Leaf1(config-router-bgp)#address-family ipv4 
Leaf1(config-router-bgp-af)#neighbor 10.1.1.1 activate 
Leaf1(config-router-bgp-af)#neighbor 10.1.2.1 activate 

Leaf2(config)#router bgp 65532
Leaf2(config-router-bgp)#neighbor 10.1.1.1 remote-as 65534
Leaf2(config-router-bgp)#neighbor 10.1.1.1 update-source loopback 1
Leaf2(config-router-bgp)#neighbor 10.1.1.1 ebgp-multihop 
Leaf2(config-router-bgp)#neighbor 10.1.2.1 remote-as 65534
Leaf2(config-router-bgp)#neighbor 10.1.2.1 update-source loopback 1
Leaf2(config-router-bgp)#neighbor 10.1.2.1 ebgp-multihop 
Leaf2(config-router-bgp)#address-family ipv4
Leaf2(config-router-bgp-af)#neighbor 10.1.1.1 activate 
Leaf2(config-router-bgp-af)#neighbor 10.1.2.1 activate 

Leaf3(config)#router bgp 65533
Leaf3(config-router-bgp)#neighbor 10.1.1.1 remote-as 65534
Leaf3(config-router-bgp)#neighbor 10.1.1.1 update-source loopback 1
Leaf3(config-router-bgp)#neighbor 10.1.1.1 ebgp-multihop 
Leaf3(config-router-bgp)#neighbor 10.1.2.1 remote-as 65534 
Leaf3(config-router-bgp)#neighbor 10.1.2.1 update-source loopback 1 
Leaf3(config-router-bgp)#neighbor 10.1.2.1 ebgp-multihop 
Leaf3(config-router-bgp)#address-family ipv4 
Leaf3(config-router-bgp-af)#neighbor 10.1.1.1 activate 
Leaf3(config-router-bgp-af)#neighbor 10.1.2.1 activate 
```

Проверим что всё работает

```console
Leaf1#sh ip route

VRF: default
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

 O        10.1.1.1/32 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.1.2/32 [120/20] via 10.1.1.5, Ethernet1
 C        10.1.1.4/30 is directly connected, Ethernet1
 O        10.1.1.8/30 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.1.12/30 [120/20] via 10.1.1.5, Ethernet1
 O        10.1.2.1/32 [120/20] via 10.1.2.5, Ethernet2
 O        10.1.2.2/32 [120/20] via 10.1.2.5, Ethernet2
 C        10.1.2.4/30 is directly connected, Ethernet2
 O        10.1.2.8/30 [120/20] via 10.1.2.5, Ethernet2
 O        10.1.2.12/30 [120/20] via 10.1.2.5, Ethernet2
 C        10.1.3.1/32 is directly connected, Loopback1
 C        10.1.3.2/32 is directly connected, Loopback2
 O        10.1.4.1/32 [120/30] via 10.1.1.5, Ethernet1
                               via 10.1.2.5, Ethernet2
 B E      10.1.4.2/32 [200/0] via 10.1.1.5, Ethernet1
                              via 10.1.2.5, Ethernet2
 O        10.1.5.1/32 [120/30] via 10.1.1.5, Ethernet1
                               via 10.1.2.5, Ethernet2
 B E      10.1.5.2/32 [200/0] via 10.1.1.5, Ethernet1
                              via 10.1.2.5, Ethernet2

Leaf1#ping 10.1.4.2
PING 10.1.4.2 (10.1.4.2) 72(100) bytes of data.
80 bytes from 10.1.4.2: icmp_seq=1 ttl=63 time=28.8 ms
80 bytes from 10.1.4.2: icmp_seq=2 ttl=63 time=43.1 ms
80 bytes from 10.1.4.2: icmp_seq=3 ttl=63 time=44.8 ms
80 bytes from 10.1.4.2: icmp_seq=4 ttl=63 time=39.0 ms
80 bytes from 10.1.4.2: icmp_seq=5 ttl=63 time=50.0 ms

--- 10.1.4.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 88ms
rtt min/avg/max/mdev = 28.876/41.205/50.084/7.108 ms, pipe 3, ipg/ewma 22.178/35.366 ms
Leaf1#sh ip bgp summary 
BGP summary information for VRF default
Router identifier 10.1.3.1, local AS number 65531
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.1         4  65534             23        38    0    0 00:11:55 Estab   2      2
  10.1.2.1         4  65534             21        27    0    0 00:04:10 Estab   2      2
  10.1.4.1         4  65532           3032      3040    0    0    1d22h Estab   2      2
  10.1.5.1         4  65533           1771      1777    0    0    1d02h Estab   2      2
Leaf1#sh ip bgp installed 
BGP routing table information for VRF default
Router identifier 10.1.3.1, local AS number 65531
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network                Next Hop              Metric  LocPref Weight Path
 * >     10.1.3.2/32            -                     0       0       -      i
 * >     10.1.4.2/32            10.1.4.1              0       100     0      65532 i
 * >     10.1.5.2/32            10.1.5.1              0       100     0      65533 i
```