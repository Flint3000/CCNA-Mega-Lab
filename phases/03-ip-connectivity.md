# Phase 03: IP Connectivity

## Ziel
Alle Netze beider Standorte sind untereinander und Richtung Internet erreichbar.

## CCNA-Bezug
3.1 bis 3.4 (Routing-Tabelle, statische Routen, OSPFv2), 1.6 und 1.7 (IPv4-Adressierung)

## Entscheidungen
- **Adressraum:** 10.0.0.0/8 (privat, RFC 1918), pro Standort ein /16: Office A 10.1.0.0/16, Office B 10.2.0.0/16. So genügt je ein Eintrag pro Standort in Routing und ACLs.
- **VLANs:** je ein /24, die dritte Stelle ist die VLAN-ID (VLAN 10 = 10.1.10.0/24). Das Gateway ist immer die .1 im jeweiligen Subnetz. Das Server-Netz hat kein VLAN: 10.2.200.0/24.
- **Im /24:** .1 bis .19 statisch (Gateway, Geräte), .20 bis .254 DHCP. Das Management-VLAN ist komplett statisch.
- **Link-Netze:** /30 aus 10.0.0.0/24 (zwei nutzbare Adressen, kein Verschnitt). Loopbacks 10.255.0.x/32 als Router-ID.
- **Internet-Link:** 203.0.113.0/30 (Dokumentationsbereich) als Platzhalter für öffentliche Adressen, damit NAT/PAT realistisch bleibt.
- **OSPFv2, Area 0** auf R0, R-B1, CSW-A1, DSW-A1 und DSW-A2.
- **Passive Interfaces:** `passive-interface default`, aktiv nur auf den Links zwischen den Routern und Multilayer-Switches. SVIs, Subinterfaces und Loopbacks bleiben passiv und werden trotzdem angekündigt.
- **Default-Route:** Eine statische Default-Route auf R0 Richtung ISP, die R0 als ASBR mit `default-information originate` an alle anderen weitergibt.
- **Office B per Router-on-a-Stick:** Ein Trunk zwischen R-B1 und ASW-B1 mit einem Subinterface pro VLAN (50 und 99) und Native-VLAN 999.
- **OSPF-Netzwerktyp point-to-point** auf den /30-Links statt des Standards `broadcast`: Pro Link gibt es nur zwei Router, also ist keine DR/BDR-Wahl nötig, und die Nachbarschaft kommt schneller zustande.

## Konfiguration

Adressierung (Beispiele, Adressen laut [IP-Plan](../docs/ip-plan.md)):
```
! Routed Port (CSW-A1)
interface f0/2
 ip address 10.0.0.9 255.255.255.252

! Gateway-SVI (DSW-A2)
ip routing
interface vlan 30
 ip address 10.1.30.1 255.255.255.0

! Loopback (CSW-A1)
interface loopback 0
 ip address 10.255.0.3 255.255.255.255

! Default-Route Richtung ISP (R0)
ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

OSPF auf R0:
```
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.0.4 0.0.0.3 area 0
 network 10.255.0.1 0.0.0.0 area 0
 default-information originate
 passive-interface default
 no passive-interface g0/0
 no passive-interface g0/1
interface range g0/0 - 1
 ip ospf network point-to-point
```

OSPF auf DSW-A2 (die SVIs werden angekündigt, bleiben aber passiv):
```
router ospf 1
 network 10.0.0.12 0.0.0.3 area 0
 network 10.1.30.0 0.0.0.255 area 0
 network 10.1.40.0 0.0.0.255 area 0
 network 10.1.99.0 0.0.0.255 area 0
 network 10.255.0.5 0.0.0.0 area 0
 passive-interface default
 no passive-interface f0/5
```

Netzwerktyp auf den Routed Ports der DSW (Beispiel DSW-A1):
```
interface f0/2
 ip ospf network point-to-point
```

Office B: Router-on-a-Stick auf R-B1. Die VLANs werden nur auf den Switches angelegt, der Router braucht kein VLAN, die VLAN-ID steht nur in `encapsulation dot1q`:
```
interface g0/1
 no shutdown
interface g0/1.50
 encapsulation dot1q 50
 ip address 10.2.50.1 255.255.255.0
interface g0/1.99
 encapsulation dot1q 99
 ip address 10.2.99.1 255.255.255.0
interface g0/1.999
 encapsulation dot1q 999 native
```
ASW-B1 hat die VLANs 50, 99 und 999. Fa0/1 (PC7) ist Access in VLAN 99, die Trunks Fa0/2 (zu ASW-B2) und G0/1 (zu R-B1) erlauben 50 und 99 mit Native-VLAN 999.

## Verifikation

| Befehl | Gerät | Erwartung |
|---|---|---|
| `show ip interface brief` | CSW-A1, DSW-A2, R-B1 | Routed Ports, SVIs, Subinterfaces und Loopback up/up mit IP |
| `show ip route` | CSW-A1, R0 | Netze beider Standorte und Default-Route |
| `show ip ospf neighbor` | CSW-A1 | Nachbarn R0, DSW-A1, DSW-A2 im Zustand FULL |
| `show running-config \| section ospf` | CSW-A1 | Netzwerktyp point-to-point, passive Interfaces |
| `show vlan brief`, `show interfaces trunk` | ASW-B1, ASW-B2 | VLAN 50, 99 und 999 vorhanden, Trunks mit VLAN 50, 99 und Native 999 |
| `ping 10.2.99.100` | PC4 | Antwort über beide Standorte |

#### Adressen: `show ip interface brief`

CSW-A1 (gekürzt):
```
CSW-A1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
FastEthernet0/1        unassigned      YES unset  administratively down down 
FastEthernet0/2        10.0.0.9        YES manual up                    up 
FastEthernet0/3        unassigned      YES unset  administratively down down 
FastEthernet0/4        unassigned      YES unset  administratively down down 
FastEthernet0/5        10.0.0.13       YES manual up                    up 
[...]
GigabitEthernet0/1     10.0.0.6        YES manual up                    up 
GigabitEthernet0/2     unassigned      YES unset  administratively down down 
Loopback0              10.255.0.3      YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

DSW-A2 (gekürzt):
```
DSW-A2#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
Port-channel1          unassigned      YES unset  up                    up 
FastEthernet0/1        unassigned      YES unset  up                    up 
FastEthernet0/2        unassigned      YES unset  up                    up 
FastEthernet0/3        unassigned      YES unset  up                    up 
FastEthernet0/4        unassigned      YES unset  up                    up 
FastEthernet0/5        10.0.0.14       YES manual up                    up 
[...]
Loopback0              10.255.0.5      YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down 
Vlan30                 10.1.30.1       YES manual up                    up 
Vlan40                 10.1.40.1       YES manual up                    up 
Vlan99                 10.1.99.2       YES manual up                    up
```

R-B1 (Subinterfaces für Router-on-a-Stick):
```
R-B1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     10.0.0.2        YES manual up                    up 
GigabitEthernet0/1     unassigned      YES unset  up                    up 
GigabitEthernet0/1.50  10.2.50.1       YES manual up                    up 
GigabitEthernet0/1.99  10.2.99.1       YES manual up                    up 
GigabitEthernet0/1.999 unassigned      YES unset  up                    up 
GigabitEthernet0/2     10.2.200.1      YES manual up                    up 
Loopback0              10.255.0.2      YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down 
```

#### Routing-Tabellen: `show ip route`

CSW-A1 (Default-Route per OSPF von R0, Netze beider Standorte):
```
CSW-A1#show ip route
Gateway of last resort is 10.0.0.5 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 17 subnets, 3 masks
O       10.0.0.0/30 [110/2] via 10.0.0.5, 00:06:08, GigabitEthernet0/1
C       10.0.0.4/30 is directly connected, GigabitEthernet0/1
C       10.0.0.8/30 is directly connected, FastEthernet0/2
C       10.0.0.12/30 is directly connected, FastEthernet0/5
O       10.1.10.0/24 [110/2] via 10.0.0.10, 00:06:08, FastEthernet0/2
O       10.1.20.0/24 [110/2] via 10.0.0.10, 00:06:08, FastEthernet0/2
O       10.1.30.0/24 [110/2] via 10.0.0.14, 00:06:08, FastEthernet0/5
O       10.1.40.0/24 [110/2] via 10.0.0.14, 00:06:08, FastEthernet0/5
O       10.1.99.0/24 [110/2] via 10.0.0.10, 00:06:08, FastEthernet0/2
                     [110/2] via 10.0.0.14, 00:06:08, FastEthernet0/5
O       10.2.50.0/24 [110/3] via 10.0.0.5, 00:06:08, GigabitEthernet0/1
O       10.2.99.0/24 [110/3] via 10.0.0.5, 00:06:08, GigabitEthernet0/1
O       10.2.200.0/24 [110/3] via 10.0.0.5, 00:06:08, GigabitEthernet0/1
O       10.255.0.1/32 [110/2] via 10.0.0.5, 00:06:08, GigabitEthernet0/1
O       10.255.0.2/32 [110/3] via 10.0.0.5, 00:06:08, GigabitEthernet0/1
C       10.255.0.3/32 is directly connected, Loopback0
O       10.255.0.4/32 [110/2] via 10.0.0.10, 00:06:08, FastEthernet0/2
O       10.255.0.5/32 [110/2] via 10.0.0.14, 00:06:08, FastEthernet0/5
O*E2 0.0.0.0/0 [110/1] via 10.0.0.5, 00:06:08, GigabitEthernet0/1
```

R0 (statische Default-Route Richtung ISP):
```
R0#show ip route
Gateway of last resort is 203.0.113.1 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 19 subnets, 3 masks
C       10.0.0.0/30 is directly connected, GigabitEthernet0/0
L       10.0.0.1/32 is directly connected, GigabitEthernet0/0
C       10.0.0.4/30 is directly connected, GigabitEthernet0/1
L       10.0.0.5/32 is directly connected, GigabitEthernet0/1
O       10.0.0.8/30 [110/2] via 10.0.0.6, 00:06:41, GigabitEthernet0/1
O       10.0.0.12/30 [110/2] via 10.0.0.6, 00:06:41, GigabitEthernet0/1
O       10.1.10.0/24 [110/3] via 10.0.0.6, 00:06:31, GigabitEthernet0/1
O       10.1.20.0/24 [110/3] via 10.0.0.6, 00:06:31, GigabitEthernet0/1
O       10.1.30.0/24 [110/3] via 10.0.0.6, 00:06:31, GigabitEthernet0/1
O       10.1.40.0/24 [110/3] via 10.0.0.6, 00:06:31, GigabitEthernet0/1
O       10.1.99.0/24 [110/3] via 10.0.0.6, 00:06:31, GigabitEthernet0/1
O       10.2.50.0/24 [110/2] via 10.0.0.2, 00:06:41, GigabitEthernet0/0
O       10.2.99.0/24 [110/2] via 10.0.0.2, 00:06:41, GigabitEthernet0/0
O       10.2.200.0/24 [110/2] via 10.0.0.2, 00:06:41, GigabitEthernet0/0
C       10.255.0.1/32 is directly connected, Loopback0
O       10.255.0.2/32 [110/2] via 10.0.0.2, 00:06:41, GigabitEthernet0/0
O       10.255.0.3/32 [110/2] via 10.0.0.6, 00:06:41, GigabitEthernet0/1
O       10.255.0.4/32 [110/3] via 10.0.0.6, 00:06:31, GigabitEthernet0/1
O       10.255.0.5/32 [110/3] via 10.0.0.6, 00:06:31, GigabitEthernet0/1
     203.0.113.0/24 is variably subnetted, 2 subnets, 2 masks
C       203.0.113.0/30 is directly connected, GigabitEthernet0/2
L       203.0.113.2/32 is directly connected, GigabitEthernet0/2
S*   0.0.0.0/0 [1/0] via 203.0.113.1
```

#### OSPF-Nachbarn: `show ip ospf neighbor`

CSW-A1 (alle drei Nachbarn im Zustand FULL):
```
CSW-A1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.255.0.4        0   FULL/  -        00:00:37    10.0.0.10       FastEthernet0/2
10.255.0.5        0   FULL/  -        00:00:37    10.0.0.14       FastEthernet0/5
10.255.0.1        0   FULL/  -        00:00:37    10.0.0.5        GigabitEthernet0/1
```

#### OSPF-Konfiguration: `show running-config | section ospf`

CSW-A1:
```
CSW-A1#show running-config | section ospf
 ip ospf network point-to-point
 ip ospf priority 1
 ip ospf network point-to-point
 ip ospf priority 1
 ip ospf network point-to-point
 ip ospf priority 1
router ospf 1
 log-adjacency-changes
 passive-interface default
 no passive-interface FastEthernet0/2
 no passive-interface FastEthernet0/5
 no passive-interface GigabitEthernet0/1
 network 10.0.0.4 0.0.0.3 area 0
 network 10.0.0.8 0.0.0.3 area 0
 network 10.0.0.12 0.0.0.3 area 0
 network 10.255.0.3 0.0.0.0 area 0
```

#### Office B: `show vlan brief` und `show interfaces trunk`

ASW-B1 (VLAN 99 mit PC7 an Fa0/1, VLAN 50 und 999 ohne Access-Ports; gekürzt):
```
ASW-B1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/3, Fa0/4, Fa0/5, Fa0/6
                                                [...]
                                                Fa0/23, Fa0/24, Gig0/2
50   PUBLIC                           active    
99   IT-MGMT                          active    Fa0/1
999  NATIVE                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```

ASW-B1 (Trunks zu ASW-B2 und R-B1):
```
ASW-B1#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/2       on           802.1q         trunking      999
Gig0/1      on           802.1q         trunking      999

Port        Vlans allowed on trunk
Fa0/2       50,99
Gig0/1      50,99

Port        Vlans allowed and active in management domain
Fa0/2       50,99
Gig0/1      50,99

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/2       50,99
Gig0/1      50,99
```

ASW-B2 (VLAN 99 mit dem Access-Port Gig0/2, VLAN 50 und 999 ohne Access-Ports; gekürzt):
```
ASW-B2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/3, Fa0/4, Fa0/5
                                                [...]
                                                Fa0/22, Fa0/23, Fa0/24
50   PUBLIC                           active    
99   IT-MGMT                          active    Gig0/2
999  NATIVE                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```

ASW-B2 (Trunks zu ASW-B1 und WLC):
```
ASW-B2#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      999
Gig0/1      on           802.1q         trunking      999

Port        Vlans allowed on trunk
Fa0/1       50,99
Gig0/1      50,99

Port        Vlans allowed and active in management domain
Fa0/1       50,99
Gig0/1      50,99

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       50,99
Gig0/1      50,99
```

#### Erreichbarkeit: Ping PC4 → PC7

PC4 (10.1.99.100, Office A) erreicht PC7 (10.2.99.100, Office B) über Distribution, Core und beide Router:
```
C:\>ping 10.2.99.100

Pinging 10.2.99.100 with 32 bytes of data:

Reply from 10.2.99.100: bytes=32 time=1ms TTL=124
Reply from 10.2.99.100: bytes=32 time<1ms TTL=124
Reply from 10.2.99.100: bytes=32 time<1ms TTL=124
Reply from 10.2.99.100: bytes=32 time<1ms TTL=124
```

## Probleme und Learnings
- **`default-information originate` Die Verbindung von R0 zum ISP muss zuerst stehen, bevor die konfigurierte default route per OSPF kommuniziert wird.
- **Die PT-Cloud braucht ein zweites Ethernet-Modul,** sonst lässt sich nur ein Router anschließen (ISP oder R0). Im Reiter „Physical“ das Gerät ausschalten, das Modul `PT-CLOUD-NM-1CGE` (1-Port GigabitEthernet) einsetzen und das Gerät wieder anschalten.
- **Passive SVIs erscheinen trotzdem in `show ip ospf interface`.** Sie stehen dort mit „No Hellos (Passive interface)“, das ist der Beleg, dass sie passiv sind.
- **Bei ROAS müssen die VLANs nicht auf dem Router existieren.** Der Router (2911 ohne Switch-Modul) arbeitet nur auf Layer 3 und ordnet Frames anhand des 802.1Q-Tags dem Subinterface zu (`encapsulation dot1q <ID>`, beim Native-VLAN mit `native`). Dass `show vlan brief` in Packet Tracer trotzdem die Standard-VLANs (1, 1002 bis 1005) anzeigt, ist für die Funktion ohne Bedeutung.
- **Der Access Point liegt im Management-VLAN 99, nicht im Client-VLAN 50.** Ein Lightweight AP verbindet sich über CAPWAP mit dem WLC, der ebenfalls in VLAN 99 liegt. Sein Access-Port (ASW-B2 G0/2) braucht deshalb nur VLAN 99. Der Verkehr der WLAN-Clients wird zum WLC getunnelt, und der WLC setzt ihn über den Trunk (ASW-B2 G0/1, VLANs 50 und 99) in VLAN 50 ab.
