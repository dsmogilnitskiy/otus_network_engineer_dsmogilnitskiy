## Дз3 Построение Underlay сети(IS-IS)
### План работ:
1. Добавить L3 схему сети
2. Добавить таблицу с ip пространством
3. Увеличить административную дистанцию протокола OSPF
4. Включить на всех устройствах процесс IS-IS
5. Настроить на всех устройствах DIS, IS-type и isis ciruit-type и добавить все интерфейсы в IS-IS
6. Настроить аутентификацию IS-IS

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

### 3. Увеличить административную дистанцию протокола OSPF
Административную дистанцию увеличиваем до 120 с целью включения в эксплуатацию протокола IS-IS:

```console
Spine1#conf t
Spine1(config)#router ospf 99
Spine1(config-router-ospf)#distance ospf intra-area 120

Spine2#conf t
Spine2(config)#router ospf 99
Spine2(config-router-ospf)#distance ospf intra-area 120

Leaf1# conf t
Leaf1(config)#router ospf 99
Leaf1(config-router-ospf)#distance ospf intra-area 120

Leaf2#conf t
Leaf2(config)#router ospf 99
Leaf2(config-router-ospf)#distance ospf intra-area 120

Leaf3#conf t
Leaf3(config)#router ospf 99
Leaf3(config-router-ospf)#distance ospf intra-area 120
```
Проверка:
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

### 4. Включить на всех устройствах процесс IS-IS
```console
Spine1#conf t
Spine1(config)#router isis 99

Spine2#conf t
Spine2(config)#router isis 99

Leaf1#conf t
Leaf1(config)#router isis 99

Leaf2#conf t
Leaf2(config)#router isis 99

Leaf3#conf t
Leaf3(config)#router isis 99
```

### 5. Настроить на всех устройствах DIS, IS-type и isis ciruit-type и добавить все интерфейсы в IS-IS
Настраиваем DIS по логике 49.номер_зоны.айпи_лупбека.00

```console
Spine1#conf t
Spine1(config)#router isis 99
Spine1(config-router-isis)#net 49.0001.0100.0100.1001.00
Spine1(config-router-isis)#router-id ipv4 10.1.1.1
Spine1(config-router-isis)#is-type level-1-2
Spine1(config-router-isis)#address-family ipv4 unicast
Spine1(config)#interface Ethernet1
Spine1(config-if-Et1)#isis enable 99
Spine1(config-if-Et1)#isis circuit-type level-1
Spine1(config)#int ethernet 2
Spine1(config-if-Et2)#isis enable 99
Spine1(config-if-Et2)#isis circuit-type level-1
Spine1(config-if-Et2)#int ethernet 3
Spine1(config-if-Et3)#isis enable 99
Spine1(config-if-Et3)#isis circuit-type level-1
Spine1(config)#int loopback 1
Spine1(config-if-Lo1)#isis enable 99
Spine1(config-if-Lo1)#isis circuit-type level-1
Spine1(config)#int loopback 2
Spine1(config-if-Lo2)#isis enable 99
Spine1(config-if-Lo2)#isis circuit-type level-1

Spine2#conf t
Spine2(config)#router isis 99
Spine2(config-router-isis)#net 49.0001.0100.0100.2001.00
Spine2(config-router-isis)#router-id ipv4 10.1.2.1
Spine2(config-router-isis-af)#is-type level-1-2
Spine2(config-router-isis)#address-family ipv4 unicast
Spine2(config)#interface ethernet 1
Spine2(config-if-Et1)#isis enable 99
Spine2(config-if-Et1)#isis circuit-type level-1
Spine2(config)#interface ethernet 2
Spine2(config-if-Et2)#isis enable 99
Spine2(config-if-Et2)#isis circuit-type level-1
Spine2(config)#interface ethernet 3
Spine2(config-if-Et3)#isis enable 99
Spine2(config-if-Et3)#isis circuit-type level-1
Spine2(config-if-Et2)#interface loopback 1
Spine2(config-if-Lo1)#isis enable 99
Spine2(config-if-Lo1)#isis circuit-type level-1
Spine2(config-if-Lo1)#interface loopback 2
Spine2(config-if-Lo2)#isis enable 99
Spine2(config-if-Lo2)#isis circuit-type level-1

Leaf1#conf t
Leaf1(config)#router isis 99
Leaf1(config-router-isis)#net 49.0001.0100.0100.3001.00
Leaf1(config-router-isis)#router-id ipv4 10.1.1.1
Leaf1(config-router-isis)#is-type level-1-2
Leaf1(config-router-isis)#address-family ipv4 unicast
Leaf1(config)#interface Ethernet1
Leaf1(config-if-Et1)#isis enable 99
Leaf1(config-if-Et1)#isis circuit-type level-1
Leaf1(config)#int ethernet 2
Leaf1(config-if-Et2)#isis enable 99
Leaf1(config-if-Et2)#isis circuit-type level-1
Leaf1(config)#int loopback 1
Leaf1(config-if-Lo1)#isis enable 99
Leaf1(config-if-Lo1)#isis circuit-type level-1
Leaf1(config)#int loopback 2
Leaf1(config-if-Lo2)#isis enable 99
Leaf1(config-if-Lo2)#isis circuit-type level-1

Leaf2#conf t
Leaf2(config)#router isis 99
Leaf2(config-router-isis)#net 49.0001.0100.0100.4001.00
Leaf2(config-router-isis)#router-id ipv4 10.1.4.1
Leaf2(config-router-isis)#is-type level-1-2
Leaf2(config-router-isis)#address-family ipv4 unicast
Leaf2(config-router-isis)#int eth1
Leaf2(config-if-Et1)#isis enable 99
Leaf2(config-if-Et1)#isis circuit-type level-1
Leaf2(config-if-Et1)#int ethernet 2
Leaf2(config-if-Et2)#isis enable 99
Leaf2(config-if-Et2)#isis circuit-type level-1
Leaf2(config)#int loopback 1
Leaf2(config-if-Lo1)#isis enable 99
Leaf2(config-if-Lo1)#isis circuit-type level-1
Leaf2(config-if-Lo1)#int loopback 2
Leaf2(config-if-Lo2)#isis enable 99
Leaf2(config-if-Lo2)#isis circuit-type level-1

Leaf3#conf t
Leaf3(config)#router isis 99
Leaf3(config-router-isis)#net 49.0001.0100.0100.5001.00
Leaf3(config-router-isis)#router-id ipv4 10.1.5.1
Leaf3(config-router-isis)#is-type level-1-2
Leaf3(config-router-isis)#address-family ipv4 unicast 
Leaf3(config-router-isis-af)#int ethernet 1
Leaf3(config-if-Et1)#isis enable 99
Leaf3(config-if-Et1)#isis circuit-type level-1
Leaf3(config-if-Et1)#int ethernet 2
Leaf3(config-if-Et2)#isis enable 99
Leaf3(config-if-Et2)#isis circuit-type level-1
Leaf3(config)#int loopback 1
Leaf3(config-if-Lo1)#isis enable 99
Leaf3(config-if-Lo1)#isis circuit-type level-1
Leaf3(config-if-Lo1)#int loopback 2
Leaf3(config-if-Lo2)#isis enable 99
Leaf3(config-if-Lo2)#isis circuit-type level-1
```
Проверка:

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
 I L1     10.1.2.1/32 [115/30] via 10.1.1.6, Ethernet1
                               via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 I L1     10.1.2.2/32 [115/30] via 10.1.1.6, Ethernet1
                               via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 I L1     10.1.2.4/30 [115/20] via 10.1.1.6, Ethernet1
 I L1     10.1.2.8/30 [115/20] via 10.1.1.10, Ethernet2
 I L1     10.1.2.12/30 [115/20] via 10.1.1.14, Ethernet3
 I L1     10.1.3.1/32 [115/20] via 10.1.1.6, Ethernet1
 I L1     10.1.3.2/32 [115/20] via 10.1.1.6, Ethernet1
 I L1     10.1.4.1/32 [115/20] via 10.1.1.10, Ethernet2
 I L1     10.1.4.2/32 [115/20] via 10.1.1.10, Ethernet2
 I L1     10.1.5.1/32 [115/20] via 10.1.1.14, Ethernet3
 I L1     10.1.5.2/32 [115/20] via 10.1.1.14, Ethernet3
```
```console
Spine1#sh isis database   
IS-IS Instance: 99 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Spine1.00-00                 23   5760  1193    159 L2 <>
    Spine1.1e-00                  7  55672  1193     51 L2 <>
    Spine1.1f-00                  3  23529  1193     51 L2 <>
    Spine1.20-00                  2  54878  1193     51 L2 <>
    Spine2.00-00                 12  23210  1195    135 L2 <>
    Spine2.1f-00                  3  64297  1195     51 L2 <>
    Leaf1.00-00                  23  35325  1194    134 L2 <>
    Leaf1.1f-00                   3  39817  1194     51 L2 <>
    Leaf2.00-00                  10  17975  1194    134 L2 <>
    Leaf3.00-00                   7  28736  1194    123 L2 <>
  IS-IS Level 2 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Spine1.00-00                 34  31105  1195    201 L2 <>
```
```console
Spine1#sh isis neighbors 
 Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id          
99        default  Leaf1            L1   Ethernet1          50:0:0:d5:5d:c0   UP    24          Spine1.1e           
99        default  Leaf2            L1   Ethernet2          50:0:0:3:37:66    UP    23          Spine1.1f           
99        default  Leaf3            L1   Ethernet3          50:0:0:15:f4:e8   UP    30          Spine1.20   
```

### 6. Настроить аутентификацию IS-IS.
Для уменьшения вывода рассмотрим на связке Spine1-Leaf1

```console
Spine1#conf t
Spine1(config)#interface Ethernet1
Spine1(config-if-Et1)#isis network point-to-point
Spine1(config-if-Et1)#isis authentication mode md5
Spine1(config-if-Et1)#isis authentication key 7 TB1Je5TPRlIB2skrgLFR+A==
```

Вывод таблицы маршрутизации, видим что Ethernet1 больше нет
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
 I L1     10.1.2.1/32 [115/30] via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 I L1     10.1.2.2/32 [115/30] via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 I L1     10.1.2.4/30 [115/30] via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 I L1     10.1.2.8/30 [115/20] via 10.1.1.10, Ethernet2
 I L1     10.1.2.12/30 [115/20] via 10.1.1.14, Ethernet3
 I L1     10.1.3.1/32 [115/40] via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 I L1     10.1.3.2/32 [115/40] via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 I L1     10.1.4.1/32 [115/20] via 10.1.1.10, Ethernet2
 I L1     10.1.4.2/32 [115/20] via 10.1.1.10, Ethernet2
 I L1     10.1.5.1/32 [115/20] via 10.1.1.14, Ethernet3
 I L1     10.1.5.2/32 [115/20] via 10.1.1.14, Ethernet3
```
Аналогично настраиваем на Leaf1 и проверяем таблицу маршрутизации

```console
Leaf1#conf t
Leaf1(config)#int ethernet 1
Leaf1(config-if-Et1)#isis network point-to-point 
Leaf1(config-if-Et1)#isis authentication mode md5 
Leaf1(config-if-Et1)#isis authentication key cisco
```

Видим что Ethernet1 снова добавился в таблицу маршрутизации

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
 I L1     10.1.2.1/32 [115/30] via 10.1.1.6, Ethernet1
                               via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 I L1     10.1.2.2/32 [115/30] via 10.1.1.6, Ethernet1
                               via 10.1.1.10, Ethernet2
                               via 10.1.1.14, Ethernet3
 I L1     10.1.2.4/30 [115/20] via 10.1.1.6, Ethernet1
 I L1     10.1.2.8/30 [115/20] via 10.1.1.10, Ethernet2
 I L1     10.1.2.12/30 [115/20] via 10.1.1.14, Ethernet3
 I L1     10.1.3.1/32 [115/20] via 10.1.1.6, Ethernet1
 I L1     10.1.3.2/32 [115/20] via 10.1.1.6, Ethernet1
 I L1     10.1.4.1/32 [115/20] via 10.1.1.10, Ethernet2
 I L1     10.1.4.2/32 [115/20] via 10.1.1.10, Ethernet2
 I L1     10.1.5.1/32 [115/20] via 10.1.1.14, Ethernet3
 I L1     10.1.5.2/32 [115/20] via 10.1.1.14, Ethernet3
```
Аналогичным образом настраиваем на остальных устройствах/интерфейсах

```console
Spine1#conf t
Spine1(config)#int ethernet 2
Spine1(config-if-Et2)#isis network point-to-point
Spine1(config-if-Et2)#isis authentication mode md5
Spine1(config-if-Et2)#isis authentication key cisco
Spine1(config-if-Et2)#int ethernet 3
Spine1(config-if-Et3)#isis network point-to-point
Spine1(config-if-Et3)#isis authentication mode md5
Spine1(config-if-Et3)#isis authentication key cisco

Spine2(config)#int ethernet 1
Spine2(config-if-Et1)#isis network point-to-point 
Spine2(config-if-Et1)#isis authentication mode md5 
Spine2(config-if-Et1)#isis authentication key cisco
Spine2(config-if-Et1)#int ethernet 2
Spine2(config-if-Et2)#isis network point-to-point
Spine2(config-if-Et2)#isis authentication mode md5
Spine2(config-if-Et2)#isis authentication key cisco

Leaf1(config)#int ethernet 2
Leaf1(config-if-Et2)#isis network point-to-point
Leaf1(config-if-Et2)#isis authentication mode md5
Leaf1(config-if-Et2)#isis authentication key cisco

Leaf2(config)#int ethernet 1
Leaf2(config-if-Et1)#isis network point-to-point 
Leaf2(config-if-Et1)#isis authentication mode md5 
Leaf2(config-if-Et1)#isis authentication key cisco
Leaf2(config-if-Et1)#int ethernet 2
Leaf2(config-if-Et2)#isis network point-to-point
Leaf2(config-if-Et2)#isis authentication mode md5
Leaf2(config-if-Et2)#isis authentication key cisco

Leaf3(config)#int ethernet 1
Leaf3(config-if-Et1)#isis network point-to-point 
Leaf3(config-if-Et1)#isis authentication mode md5 
Leaf3(config-if-Et1)#isis authentication key cisco
Leaf3(config-if-Et1)#int ethernet 2
Leaf3(config-if-Et2)#isis network point-to-point
Leaf3(config-if-Et2)#isis authentication mode md5
Leaf3(config-if-Et2)#isis authentication key cisco
```

