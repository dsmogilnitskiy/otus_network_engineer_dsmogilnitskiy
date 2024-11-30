## Дз4 Построение Underlay сети(eBGP)
### План работ:
1. Добавить L3 схему сети
2. Добавить таблицу с ip пространством
3. Выпиливание протокола IS-IS
4. Исключение лупбеков на лиф коммутаторах из OSPF
4. Включить на всех устройствах eBGP
5. Настроить на всех устройствах RID
6. Поднять соседство между всеми Leaf коммутаторами
7. Настроить BFD
8. Проверить доступность всех лупбеков в BGP

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
Исключаем лупбеки из OSPF на лиф коммутаторах так как в последующем настроим BGP для анонса лупбеков лифов. Все физические интерфейсы и лупбеки спайн коммутаторов попрежнему будут находиться в OSPF.

```console
Leaf1(config)#int loopback 1
Leaf1(config-if-Lo1)#no ip ospf area 0.0.0.0
Leaf1(config-if-Lo1)#int loopback 2
Leaf1(config-if-Lo2)#no ip ospf area 0.0.0.0

Leaf2(config)#int loopback 1
Leaf2(config-if-Lo1)#no ip ospf area 0.0.0.0
Leaf2(config-if-Lo1)#int loopback 2
Leaf2(config-if-Lo2)#no ip ospf area 0.0.0.0

Leaf3(config)#interface Loopback1
Leaf3(config-if-Lo1)#no ip ospf area 0.0.0.0
Leaf3(config-if-Lo1)#interface Loopback2
Leaf3(config-if-Lo2)#no ip ospf area 0.0.0.0
```
Видим, что лупбеки пропали из таблици маршрутизации

```console
Leaf3#sh ip route

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

 O        10.1.1.1/32 [120/20] via 10.1.1.13, Ethernet1
 O        10.1.1.2/32 [120/20] via 10.1.1.13, Ethernet1
 O        10.1.1.4/30 [120/20] via 10.1.1.13, Ethernet1
 O        10.1.1.8/30 [120/20] via 10.1.1.13, Ethernet1
 C        10.1.1.12/30 is directly connected, Ethernet1
 O        10.1.2.1/32 [120/20] via 10.1.2.13, Ethernet2
 O        10.1.2.2/32 [120/20] via 10.1.2.13, Ethernet2
 O        10.1.2.4/30 [120/20] via 10.1.2.13, Ethernet2
 O        10.1.2.8/30 [120/20] via 10.1.2.13, Ethernet2
 C        10.1.2.12/30 is directly connected, Ethernet2
 C        10.1.5.1/32 is directly connected, Loopback1
 C        10.1.5.2/32 is directly connected, Loopback2
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