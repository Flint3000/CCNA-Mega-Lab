# Phase 02: Layer 2 – VLANs, Trunks, EtherChannel und Spanning Tree (Office A)

## Ziel
Alle Layer-2-Switches in Office A kennen die VLANs, die sie brauchen. Access-Ports sind den richtigen VLANs zugeordnet, die Uplinks sind als 802.1Q-Trunks mit begrenzter VLAN-Liste konfiguriert, und jeder Switch ist über das Management-VLAN 99 erreichbar. Ungenutzte Ports sind abgeschaltet, die Links zum Core sind als Routed Ports vorbereitet, und Spanning Tree läuft als Rapid PVST+ mit festen Root-Bridges pro VLAN.

## CCNA-Bezug
- 2.1 VLANs, Access-Ports, Voice-VLAN
- 2.2 Trunks, 802.1Q, Native-VLAN
- 2.4 EtherChannel mit LACP (hier: Port-Channel 1 zwischen DSW-A1 und DSW-A2)
- 2.5 Rapid PVST+: Root-Bridge, Port-Rollen und -Zustände, PortFast, BPDU Guard, Root Guard, Loop Guard

## Ausgangslage
Basiskonfiguration aus [Phase 01](01-basis.md) steht, Verkabelung laut [`../docs/port-table.md`](../docs/port-table.md). Stand dieser Phase: Office A. Office B folgt in Phase 03.

## Entscheidungen
- **VLANs nur dort, wo sie gebraucht werden:** Jeder Access-Switch bekommt nur seine Benutzer-VLANs plus VLAN 99 (Management). Die Distribution-Switches kennen alle VLANs. Kein VTP, jedes VLAN wird von Hand angelegt.
- **VLAN 99 als Management-VLAN:** Trennt die Geräteverwaltung vom Benutzerverkehr und erlaubt später SSH-Zugriff per ACL nur aus dem Admin-Netz. Jeder Switch hat eine Management-SVI mit IP und `ip default-gateway`.
- **Native-VLAN 999 auf allen Trunks:** ungenutzt, damit kein Benutzerverkehr ungetaggt über Trunks läuft (Schutz vor VLAN Hopping über doppelte Tags).
- **Erlaubte VLANs pro Trunk begrenzen** (`switchport trunk allowed vlan`): Nur die VLANs des jeweiligen Access-Switches plus VLAN 99. Beide Uplinks eines Access-Switches bekommen dieselbe Liste, damit der Ausweichweg funktioniert. Der Link zwischen den DSW führt alle VLANs, weil die Gateways auf beide DSW verteilt sind.
- **DTP abgeschaltet** (`switchport nonegotiate`): Trunks lassen sich nicht aushandeln (Schutz vor Switch Spoofing). Beide Seiten sind dafür fest auf `trunk` bzw. `access` gesetzt.
- **Ungenutzte Ports per `shutdown` deaktiviert** aus Sicherheitsgründen: Niemand kann an einem freien Port unbemerkt ein Gerät anschließen.
- **Beschreibungen an jedem genutzten Port** nach dem Muster `description ##to <Gerät>##`, damit `show interfaces status` und die Config selbsterklärend sind.
- **Port-Channel 1 zwischen DSW-A1 und DSW-A2 mit LACP** (F0/7 und F0/8, `mode active` auf beiden Seiten). Zuerst mit PAgP aufgebaut, dann auf LACP umgestellt: LACP ist herstellerunabhängig (IEEE 802.3ad) und steht in der CCNA-Themenliste (2.4), PAgP ist Cisco-proprietär.
- **Links zum Core als Routed Ports** (`no switchport`) auf CSW-A1 (F0/2, F0/5, G0/1), DSW-A1 (F0/2) und DSW-A2 (F0/5): Zwischen Distribution und Core gibt es kein Layer 2 mehr. Dadurch läuft Spanning Tree nur zwischen Access- und Distribution-Switches, und der Core braucht keine VLANs. IP-Adressen, `ip routing` und OSPF folgen in Phase 03.
- **Rapid PVST+** auf allen Switches in Office A: schnellere Umschaltung als klassisches PVST+, eine STP-Instanz pro VLAN.
- **Root-Bridge pro VLAN passend zum Gateway:** DSW-A1 für VLAN 10, 20, 99 (Priority 4096, Secondary 8192), DSW-A2 für VLAN 30, 40 und 999 (4096, Secondary 8192). Die ASW bleiben auf dem Standard. Der Verkehr eines VLANs nimmt so den kürzesten Weg zum Gateway, ohne Umweg über Po1, und beide DSW werden genutzt. Eine Ausfallsicherheit entsteht dadurch nicht: Fällt ein DSW aus, haben seine VLANs kein Gateway mehr. Dafür wäre ein FHRP wie HSRP nötig.
- **PortFast und BPDU Guard** auf allen Access-Ports, **Root Guard** auf den DSW-Ports zu den ASW, **Loop Guard** auf Po1 zwischen den DSW und auf den ASW-Uplinks zu DSW-A1/DSW-A2 (in Packet Tracer nicht verfügbar, nicht konfiguriert).


## Konfiguration

### VLAN-Matrix (Office A)

| Switch | VLANs |
|---|---|
| ASW-A1 | 10 (HR), 99 |
| ASW-A2 | 20 (ACCOUNTING), 99 |
| ASW-A3 | 99 (PC4 liegt direkt in VLAN 99) |
| ASW-A4 | 30 (LOGISTICS), 40 (VOICE), 99 |
| DSW-A1, DSW-A2 | 10, 20, 30, 40, 99 |
| CSW-A1 | keine (nur geroutete Links) |
| alle Switches mit Trunks | zusätzlich 999 (NATIVE), ohne Access-Ports |

### Trunk-Tabelle

| Gerät | Port | Gegenstelle | Erlaubte VLANs | Native |
|---|---|---|---|---|
| ASW-A1 | F0/3 | DSW-A1 F0/3 | 10, 99 | 999 |
| ASW-A1 | F0/4 | DSW-A2 F0/4 | 10, 99 | 999 |
| ASW-A2 | F0/2 | DSW-A1 F0/1 | 20, 99 | 999 |
| ASW-A2 | F0/1 | DSW-A2 F0/3 | 20, 99 | 999 |
| ASW-A3 | F0/4 | DSW-A1 F0/4 | 99 | 999 |
| ASW-A3 | F0/2 | DSW-A2 F0/1 | 99 | 999 |
| ASW-A4 | F0/5 | DSW-A1 F0/5 | 30, 40, 99 | 999 |
| ASW-A4 | F0/3 | DSW-A2 F0/2 | 30, 40, 99 | 999 |
| DSW-A1 / DSW-A2 | Po1 (F0/7, F0/8) | gegenseitig | 10, 20, 30, 40, 99 | 999 |

### Management-IPs (VLAN 99, Office A)

| Gerät | IP | Default-Gateway |
|---|---|---|
| ASW-A1 | 10.1.99.11/24 | 10.1.99.1 |
| ASW-A2 | 10.1.99.12/24 | 10.1.99.1 |
| ASW-A3 | 10.1.99.13/24 | 10.1.99.1 |
| ASW-A4 | 10.1.99.14/24 | 10.1.99.1 |
| DSW-A1 | 10.1.99.1/24 | Gateway des Netzes (Routing folgt in Phase 03) |
| DSW-A2 | 10.1.99.2/24 | – |

### Beispiel: VLANs
```
vlan 10
 name HR
vlan 20
 name ACCOUNTING
vlan 30
 name LOGISTICS
vlan 40
 name VOICE
vlan 99
 name IT-MGMT
vlan 999
 name NATIVE
```

### Beispiel: Trunk (ASW-A1, Uplink zu DSW-A1)
```
interface f0/3
 description ##to DSW-A1##
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,99
 switchport nonegotiate
```
Auf den DSW (3560) kommt vor `switchport mode trunk` zusätzlich `switchport trunk encapsulation dot1q`, auf den 2960ern entfällt der Befehl.

### Beispiel: Access-Port (ASW-A2, PC3)
```
interface f0/3
 description ##to PC3##
 switchport mode access
 switchport access vlan 20
 switchport nonegotiate
```

### Beispiel: Telefon-Port (ASW-A4, Voice-VLAN)
```
interface f0/2
 description ##to IPPh-A1##
 switchport mode access
 switchport access vlan 30
 switchport voice vlan 40
 switchport nonegotiate
```

### Beispiel: Management-SVI (ASW-A1)
```
interface vlan 99
 ip address 10.1.99.11 255.255.255.0
 no shutdown
ip default-gateway 10.1.99.1
```

### Port-Channel 1 (DSW-A1 und DSW-A2, LACP)
Auf beiden DSW identisch (Beschreibung jeweils auf die Gegenseite setzen). Reihenfolge: erst Po1 vollständig konfigurieren, danach die Mitgliedsports mit `channel-group` hinzufügen. Port-Channel und Mitgliedsports müssen in Modus, Native-VLAN, erlaubten VLANs und DTP-Einstellung übereinstimmen, sonst werden die Ports suspendiert.
```
interface port-channel 1
 description ##to DSW-A2##
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,40,99
 switchport nonegotiate
interface range f0/7 - 8
 description ##to DSW-A2##
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,40,99
 switchport nonegotiate
 channel-group 1 mode active
```
LACP-Modi: `active` und `passive` bilden zusammen einen Channel (active/active und active/passive), passive/passive nicht.

### Routed Ports zum Core (Beispiel CSW-A1)
```
interface f0/2
 description ##to DSW-A1##
 no switchport
interface f0/5
 description ##to DSW-A2##
 no switchport
interface g0/1
 description ##to R0##
 no switchport
```
Auf den DSW entsprechend je ein Port (DSW-A1 F0/2 `##to CSW-A1##`, DSW-A2 F0/5 `##to CSW-A1##`). Die IP-Adressen der Link-Netze (/30) und `ip routing` folgen in Phase 03.

### Spanning Tree
Auf allen Switches in Office A (ASW und DSW):
```
spanning-tree mode rapid-pvst
```
Root-Bridges:
```
! DSW-A1
spanning-tree vlan 10,20,99 priority 4096
spanning-tree vlan 30,40,999 priority 8192
! DSW-A2
spanning-tree vlan 30,40,999 priority 4096
spanning-tree vlan 10,20,99 priority 8192
```
PortFast und BPDU Guard auf den ASW als default command (alle Access-Ports) und nicht per Interface:
```
spanning-tree portfast default
spanning-tree portfast bpduguard default
```
Root Guard auf den DSW-Ports zu den ASW (DSW-A1: F0/1, F0/3 - 5, DSW-A2: F0/1 - 4):
```
interface range f0/1, f0/3 - 5
 spanning-tree guard root
```
Root Guard und Loop Guard lassen sich nicht auf demselben Port kombinieren, sie sind mutually exclusive.

### Ungenutzte Ports abschalten (Beispiel ASW-A1)
```
interface range f0/5 - 24, g0/1 - 2
 shutdown
```

Genutzte Ports pro Switch (nicht abschalten):

| Gerät | Genutzte Ports |
|---|---|
| ASW-A1 | F0/1, F0/2, F0/3, F0/4 |
| ASW-A2 | F0/1, F0/2, F0/3 |
| ASW-A3 | F0/1, F0/2, F0/4 |
| ASW-A4 | F0/1, F0/2, F0/3, F0/5 |
| DSW-A1 | F0/1, F0/2 (routed), F0/3, F0/4, F0/5, F0/7, F0/8 |
| DSW-A2 | F0/1, F0/2, F0/3, F0/4, F0/5 (routed), F0/7, F0/8 |
| CSW-A1 | F0/2 (routed), F0/5 (routed), G0/1 (routed) |

## Verifikation

| Befehl | Gerät | Erwartung |
|---|---|---|
| `show vlan brief` | ASW-A4, DSW-A1 | Nur die benötigten VLANs vorhanden, Access-Ports richtig zugeordnet |
| `show interfaces trunk` | DSW-A1, ASW-A1, ASW-A4 | Trunks aktiv, Native 999, erlaubte VLANs laut Tabelle, Spanning-Tree-Zustand pro Uplink |
| `show interfaces <port> switchport` | ASW-A4 (Fa0/2, Fa0/3) | Modus und VLANs richtig, „Negotiation of Trunking: Off“ |
| `show etherchannel summary` | DSW-A2 | Po1 im Zustand `SU`, Protokoll LACP, F0/7 und F0/8 als `P` |
| `show interfaces status` | ASW-A1, DSW-A1, CSW-A1 | Genutzte Ports `connected`, ungenutzte `disabled`, Beschreibung in der Spalte „Name“ |
| `show ip interface brief` | ASW-A1 | `Vlan99` up/up mit der Management-IP |
| `ping 10.1.99.1`, `.2`, `.11` bis `.14` | PC4 | Antwort (PC4 liegt in VLAN 99, gleiches Subnetz) |
| `show spanning-tree summary` | ASW-A1 | Modus `rapid-pvst`, PortFast und BPDU Guard als Standard aktiv |
| `show spanning-tree vlan 10` | DSW-A1, ASW-A1 | DSW-A1 ist Root, ASW-A1 hat Root-Port und Alternate-Port |
| `show spanning-tree vlan 30` | DSW-A2, ASW-A4 | DSW-A2 ist Root, ASW-A4 hat Root-Port und Alternate-Port |

### Ausgaben

Die langen Listen ungenutzter Ports sind gekürzt (`[...]`), die Ausgaben von `show interfaces <port> switchport` auf die relevanten Zeilen.

#### VLANs: `show vlan brief`

ASW-A4 (VLAN 30 und 40 am Telefon-Port, VLAN 99 ohne Access-Ports, VLAN 999 ohne Ports):
```
ASW-A4#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/4, Fa0/6, Fa0/7, Fa0/8
                                                [...]
                                                Gig0/1, Gig0/2
30   LOGISTICS                        active    Fa0/1, Fa0/2
40   VOICE                            active    Fa0/2
99   IT-MGMT                          active    
999  NATIVE                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```

DSW-A1 (alle VLANs, keine Access-Ports):
```
DSW-A1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/6, Fa0/9, Fa0/10, Fa0/11
                                                [...]
                                                Fa0/24, Gig0/1, Gig0/2
10   HR                               active    
20   ACCOUNTING                       active    
30   LOGISTICS                        active    
40   VOICE                            active    
99   IT-MGMT                          active    
999  NATIVE                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active  
```

#### Trunks: `show interfaces trunk`

DSW-A1 (Native 999, erlaubte VLANs laut Trunk-Tabelle, Po1 mit allen fünf VLANs):
```
DSW-A1#sh int trunk
Port        Mode         Encapsulation  Status        Native vlan
Po1         on           802.1q         trunking      999
Fa0/1       on           802.1q         trunking      999
Fa0/3       on           802.1q         trunking      999
Fa0/4       on           802.1q         trunking      999
Fa0/5       on           802.1q         trunking      999

Port        Vlans allowed on trunk
Po1         10,20,30,40,99
Fa0/1       20,99
Fa0/3       10,99
Fa0/4       99
Fa0/5       30,40,99

Port        Vlans allowed and active in management domain
Po1         10,20,30,40,99
Fa0/1       20,99
Fa0/3       10,99
Fa0/4       99
Fa0/5       30,40,99

Port        Vlans in spanning tree forwarding state and not pruned
Po1         10,20,30,40,99
Fa0/1       20,99
Fa0/3       10,99
Fa0/4       99
Fa0/5       30,40,99
```

ASW-A4:
```
ASW-A4#show int trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/3       on           802.1q         trunking      999
Fa0/5       on           802.1q         trunking      999

Port        Vlans allowed on trunk
Fa0/3       30,40,99
Fa0/5       30,40,99

Port        Vlans allowed and active in management domain
Fa0/3       30,40,99
Fa0/5       30,40,99

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/3       30,40
Fa0/5       99
```
Fa0/3 (Root-Port zu DSW-A2) trägt VLAN 30 und 40, Fa0/5 (Alternate-Port für diese VLANs, zu DSW-A1) trägt VLAN 99. Die Root-Bridges verteilen die VLANs so auf beide Uplinks.

ASW-A1:
```
ASW-A1#sh int trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/3       on           802.1q         trunking      999
Fa0/4       on           802.1q         trunking      999

Port        Vlans allowed on trunk
Fa0/3       10,99
Fa0/4       10,99

Port        Vlans allowed and active in management domain
Fa0/3       10,99
Fa0/4       10,99

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/3       10,99
Fa0/4       none
```
Fa0/3 (Root-Port zu DSW-A1) trägt VLAN 10 und 99, Fa0/4 (Alternate-Port zu DSW-A2) blockiert alle VLANs. Beide VLANs haben ihre Root-Bridge auf DSW-A1.

#### Ports: `show interfaces <port> switchport`

ASW-A4 Fa0/2 (Telefon-Port: Access VLAN 30, Voice VLAN 40, kein DTP; gekürzt):
```
ASW-A4#show interfaces f0/2 switchport
Name: Fa0/2
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Negotiation of Trunking: Off
Access Mode VLAN: 30 (LOGISTICS)
Voice VLAN: 40
[...]
```

ASW-A4 Fa0/3 (Trunk-Port: Native VLAN 999 ist angelegt, erlaubte VLANs 30, 40, 99; gekürzt):
```
ASW-A4#show interfaces f0/3 switchport
Name: Fa0/3
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: Off
Trunking Native Mode VLAN: 999 (NATIVE)
Trunking VLANs Enabled: 30,40,99
[...]
```

#### EtherChannel: `show etherchannel summary`

DSW-A2 (Po1 mit LACP, beide Mitglieder gebündelt):
```
DSW-A2#show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           LACP   Fa0/7(P) Fa0/8(P) 
```

#### Port-Status und Beschreibungen: `show interfaces status`

ASW-A1 (genutzte Ports `connected` mit Beschreibung, ungenutzte `disabled`):
```
ASW-A1#show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     ##to PC1##         connected    10         a-full  a-100 10/100BaseTX
Fa0/2     ##to PC2##         connected    10         a-full  a-100 10/100BaseTX
Fa0/3     ##to DSW-A1##      connected    trunk      a-full  a-100 10/100BaseTX
Fa0/4     ##to DSW-A2##      connected    trunk      a-full  a-100 10/100BaseTX
Fa0/5                        disabled 1          auto    auto  10/100BaseTX
[...]
Gig0/2                       disabled 1          auto    auto  10/100/1000BaseTX
```

DSW-A1 (Trunks, Port-Channel und Routed Port zum Core):
```
DSW-A1#show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Po1       ##to DSW-A2##      connected    trunk      a-full  a-100 
Fa0/1     ##to ASW-A2##      connected    trunk      a-full  a-100 10/100BaseTX
Fa0/2     ##to CSW-A1##      connected    routed     a-full  a-100 10/100BaseTX
Fa0/3     ##to ASW-A1##      connected    trunk      a-full  a-100 10/100BaseTX
Fa0/4     ##to ASW-A3##      connected    trunk      a-full  a-100 10/100BaseTX
Fa0/5     ##to ASW-A4##      connected    trunk      a-full  a-100 10/100BaseTX
Fa0/6                        disabled 1          auto    auto  10/100BaseTX
Fa0/7     ##to DSW-A2##      connected    trunk      a-full  a-100 10/100BaseTX
Fa0/8     ##to DSW-A2##      connected    trunk      a-full  a-100 10/100BaseTX
Fa0/9                        disabled 1          auto    auto  10/100BaseTX
[...]
Gig0/2                       disabled 1          auto    auto  10/100/1000BaseTX
```

CSW-A1 (nur Routed Ports in Betrieb):
```
CSW-A1#show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1                        disabled 1          auto    auto  10/100BaseTX
Fa0/2     ##to DSW-A1##      connected    routed     a-full  a-100 10/100BaseTX
Fa0/3                        disabled 1          auto    auto  10/100BaseTX
Fa0/4                        disabled 1          auto    auto  10/100BaseTX
Fa0/5     ##to DSW-A2##      connected    routed     a-full  a-100 10/100BaseTX
Fa0/6                        disabled 1          auto    auto  10/100BaseTX
[...]
Gig0/1    ##to R0##          connected    routed     a-full  a-100 10/100/1000BaseTX
Gig0/2                       disabled 1          auto    auto  10/100/1000BaseTX
```

#### Management-SVI: `show ip interface brief`

ASW-A1 (`Vlan99` mit der Management-IP, Vlan1 abgeschaltet):
```
ASW-A1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
FastEthernet0/1        unassigned      YES manual up                    up 
FastEthernet0/2        unassigned      YES manual up                    up 
FastEthernet0/3        unassigned      YES manual up                    up 
FastEthernet0/4        unassigned      YES manual up                    up 
FastEthernet0/5        unassigned      YES manual administratively down down 
[...]
GigabitEthernet0/2     unassigned      YES manual administratively down down 
Vlan1                  unassigned      YES manual administratively down down 
Vlan99                 10.1.99.11      YES manual up                    up
```

#### Erreichbarkeit: Ping von PC4 (VLAN 99, 10.1.99.100/24)

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0060.3EC1.C596
   Link-local IPv6 Address.........: FE80::260:3EFF:FEC1:C596
   IPv6 Address....................: ::
   IPv4 Address....................: 10.1.99.100
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     10.1.99.1
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-98-42-CB-02-00-60-3E-C1-C5-96
   DNS Servers.....................: ::
                                     0.0.0.0

C:\>ping 10.1.99.11

Pinging 10.1.99.11 with 32 bytes of data:

Request timed out.
Reply from 10.1.99.11: bytes=32 time<1ms TTL=255
Reply from 10.1.99.11: bytes=32 time<1ms TTL=255
Reply from 10.1.99.11: bytes=32 time<1ms TTL=255

Ping statistics for 10.1.99.11:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),

C:\>ping 10.1.99.12

Pinging 10.1.99.12 with 32 bytes of data:

Request timed out.
Reply from 10.1.99.12: bytes=32 time<1ms TTL=255
Reply from 10.1.99.12: bytes=32 time<1ms TTL=255
Reply from 10.1.99.12: bytes=32 time<1ms TTL=255

Ping statistics for 10.1.99.12:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),

C:\>ping 10.1.99.13

Pinging 10.1.99.13 with 32 bytes of data:

Request timed out.
Reply from 10.1.99.13: bytes=32 time<1ms TTL=255
Reply from 10.1.99.13: bytes=32 time<1ms TTL=255
Reply from 10.1.99.13: bytes=32 time=1ms TTL=255

Ping statistics for 10.1.99.13:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),

C:\>ping 10.1.99.14

Pinging 10.1.99.14 with 32 bytes of data:

Request timed out.
Reply from 10.1.99.14: bytes=32 time<1ms TTL=255
Reply from 10.1.99.14: bytes=32 time<1ms TTL=255
Reply from 10.1.99.14: bytes=32 time<1ms TTL=255

Ping statistics for 10.1.99.14:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
```

Ping auf das Gateway 10.1.99.1, zuerst ohne SVI auf DSW-A1 (100 % Verlust), danach mit SVI:
```
C:\>ping 10.1.99.1

Pinging 10.1.99.1 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 10.1.99.1:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

C:\>ping 10.1.99.1

Pinging 10.1.99.1 with 32 bytes of data:

Reply from 10.1.99.1: bytes=32 time<1ms TTL=255
Reply from 10.1.99.1: bytes=32 time<1ms TTL=255
Reply from 10.1.99.1: bytes=32 time<1ms TTL=255
Reply from 10.1.99.1: bytes=32 time<1ms TTL=255

Ping statistics for 10.1.99.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
```

Ping auf DSW-A2 (belegt, dass VLAN 99 über den Port-Channel läuft):
```
C:\>ping 10.1.99.2

Pinging 10.1.99.2 with 32 bytes of data:

Reply from 10.1.99.2: bytes=32 time<1ms TTL=255
Reply from 10.1.99.2: bytes=32 time<1ms TTL=255
Reply from 10.1.99.2: bytes=32 time<1ms TTL=255
Reply from 10.1.99.2: bytes=32 time<1ms TTL=255

Ping statistics for 10.1.99.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
```

#### Spanning Tree

ASW-A1 (Portfast default und BPDU Guard default enabled; Rapid PVST+ Mode)
```

ASW-A1(config)#do sh spann sum
Switch is in rapid-pvst mode
Root bridge for:
Extended system ID           is enabled
Portfast Default             is enabled
PortFast BPDU Guard Default  is enabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is disabled
UplinkFast                   is disabled
BackboneFast                 is disabled
Configured Pathcost method used is short

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0010                     1         0        0          3          4
VLAN0099                     3         0        0          1          4
VLAN0999                     3         0        0          1          4

---------------------- -------- --------- -------- ---------- ----------
4 vlans                      7         0        0          5         12
```

DSW-A1 (Root-Bridge for vlan 10; alle Ports designated)
```
DSW-A1#sh spanning-tree vlan 10
VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    4106
             Address     00D0.584A.A9B4
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4106  (priority 4096 sys-id-ext 10)
             Address     00D0.584A.A9B4
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/3            Desg FWD 19        128.3    P2p
Po1              Desg FWD 12        128.27   P2p
```

ASW-A1 (Root-Port zu DSW-A1; Alternate-Port zu DSW-A2)
```
ASW-A1(config)#do sh spann vlan 10
VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    4106
             Address     00D0.584A.A9B4
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     000C.CF32.E107
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/3            Root FWD 19        128.3    P2p
Fa0/4            Altn BLK 19        128.4    P2p
```

DSW-A2 (Root-Bridge for vlan 30; alle Ports designated)
```
DSW-A2(config)#do sh spanning vlan 30
VLAN0030
  Spanning tree enabled protocol rstp
  Root ID    Priority    4126
             Address     00E0.F700.8A3E
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4126  (priority 4096 sys-id-ext 30)
             Address     00E0.F700.8A3E
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Po1              Desg FWD 12        128.27   P2p
```

ASW-A4 (Root-Port zu DSW-A2; Alternate-Port zu DSW-A1)
```
ASW-A4(config)#do sh spann vlan 30
VLAN0030
  Spanning tree enabled protocol rstp
  Root ID    Priority    4126
             Address     00E0.F700.8A3E
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32798  (priority 32768 sys-id-ext 30)
             Address     0002.4AC9.8EB3
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/3            Root FWD 19        128.3    P2p
Fa0/5            Altn BLK 19        128.5    P2p
```


## Probleme und Learnings
- **Router und Switches unterscheiden sich beim Standardzustand der Ports:** Router-Interfaces sind standardmäßig `shutdown` und müssen mit `no shutdown` aktiviert werden. Switch-Ports sind dagegen standardmäßig aktiv.
- **Ein Access-Port gehört genau einem VLAN.** Kommalisten wie `20,99` sind nur bei `switchport trunk allowed vlan` erlaubt. Bei `switchport access vlan` gibt es einen Syntaxfehler.
- **Trunk-Ports erscheinen nicht in `show vlan brief`.** Die Ausgabe listet nur Access-Ports. Die VLANs auf Trunks zeigt `show interfaces trunk`.
- **Der 2960 braucht kein `switchport trunk encapsulation dot1q`,** weil er nur 802.1Q kann. Der 3560 unterstützt mehrere Verfahren und verlangt den Befehl.
- **`switchport trunk native vlan 999` legt VLAN 999 nicht an.** In der Packet-Tracer-Ausgabe von `show vlan brief` fehlte VLAN 999, obwohl es als Native-VLAN auf den Trunks stand, und `show interfaces <port> switchport` meldete `Trunking Native Mode VLAN: 999 (Inactive)`. Erst nach `vlan 999` und `name NATIVE` erscheint `999 (NATIVE)`.
- **Ein PC erscheint nicht in CDP,** seine Position zeigt `show mac address-table`.
- **`show ip interface brief` zeigt keine Interface-Beschreibungen.** Dafür braucht es `show interfaces status` oder `show interfaces description`. Der Eindruck, die Beschreibungen seien nicht gespeichert, war ein Fehler bei der Wahl des Befehls.
- **Running-Config und Startup-Config sind getrennt.** Ohne `write memory` (`wr`) geht die Konfiguration bei einem Neustart verloren. Auch die `.pkt`-Datei regelmäßig speichern.
- **`ip default-gateway` braucht keine Maske.** Eine Maske gehört zu einem Netz (Interface-Adresse, Zielnetz einer Route). Das Gateway ist eine einzelne Adresse im eigenen Subnetz, ebenso ein Next-Hop.
- **Jedes Subnetz hat sein eigenes Gateway (die .1 in diesem Subnetz).** Die Access-Switches liegen mit ihrer Management-SVI in VLAN 99 und nutzen deshalb 10.1.99.1. Die PCs nutzen das Gateway ihres Benutzer-VLANs (z. B. 10.1.10.1).
- **Wechsel von PAgP auf LACP:**
  - Po1 zu löschen reicht nicht. Auf den Mitgliedsports bleibt die alte Zeile `channel-group 1 mode desirable` stehen, bis sie mit `no channel-group` entfernt wird (notfalls `default interface <Port>`).
  - Nach `default interface` sind alle Einstellungen des Ports weg und müssen identisch zu Po1 neu gesetzt werden.
  - Die Meldungen der Ports nennen die Abweichung: „vlan mask is different“ (erlaubte VLAN-Liste weicht ab) und „dtp nonegotiate is 0“ (DTP-Einstellung weicht ab).
  - PAgP und LACP lassen sich nicht mischen.
  - Zustände in `show etherchannel summary`: `s` suspended, `I` stand-alone, `P` gebündelt, `SD` heißt Layer 2, aber down (kein Mitglied gebündelt).
  - Am Ende half ein Neustart von Packet Tracer. Die Konfiguration war vorher schon korrekt, vermutlich hing die Simulation in einem alten Zustand.
- **Ein PC braucht eine IP-Adresse und Maske, bevor ein Ping funktioniert.** Bei der Verifikation fiel auf, dass PC4 (IT-MGMT) noch keine IP hatte, deshalb liefen die Pings ins Leere. Die Switch-Konfiguration war in Ordnung. VLANs und Trunks sind Layer 2, ein Ping ist ein IP-Paket.
- **Die Gateway-Adresse existiert erst, wenn die SVI da ist.** Ich hatte die SVI für VLAN 99 auf DSW-A1 vergessen, deshalb schlug der Ping von PC4 auf das Default-Gateway 10.1.99.1 fehl, während alle anderen Management-IPs im selben Subnetz erreichbar waren.
- **Loop Guard in Packet Tracer nicht verfügbar** und deshalb weder auf po1 noch auf den ASW Uplinks konfiguriert.
- **Root Guard gibt es nur pro Interface.** PortFast und BPDU Guard lassen sich global für alle Access-Ports aktivieren (`spanning-tree portfast default`, `spanning-tree portfast bpduguard default`). Root Guard dagegen nur im config-if mode (`spanning-tree guard root`).
- **Root-Bridge per Priorität statt `root primary/secondary`.** Die Priorität wird direkt gesetzt (`spanning-tree vlan 10,20,99 priority 4096`). Der Befehl `root primary` bzw. `root secondary` ist nur eine Kurzform dafür und wäre zusätzlich überflüssig, weil beide dieselbe Priorität setzen und sich überschreiben würden. Die Root-Bridge bekommt 4096 und die Secondary 8192, nicht 0. So bleibt Spielraum nach unten: Ändert sich die Topologie, kann ein anderer Switch mit einer niedrigeren Priorität (0) Root werden, ohne dass die bestehenden Werte angepasst werden müssen.
- **Erlaubte VLAN-Liste muss auf beiden Seiten eines Trunks gleich sein.** Auf DSW-A2 Fa0/2 fehlte VLAN 40, obwohl ASW-A4 es erlaubte. Symptom: ASW-A4 hatte für VLAN 40 keinen Root-Port zu DSW-A2 und erreichte die Root-Bridge nur umständlich über DSW-A1 und Po1 (Root-Cost 31 statt 19), und im dritten Block von `show interfaces trunk` stand VLAN 40 auf beiden Uplinks. Gefunden mit `show spanning-tree vlan 40` auf ASW-A4, DSW-A1 und DSW-A2. Behoben durch Ergänzen von VLAN 40 auf DSW-A2 Fa0/2.

