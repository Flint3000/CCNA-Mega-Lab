# IP-Plan

## Adressierungsschema
- Privater Adressraum 10.0.0.0/8, pro Standort ein /16: Office A 10.1.0.0/16, Office B 10.2.0.0/16.
- Jedes VLAN ist ein /24 (255.255.255.0). Die dritte Stelle ist die VLAN-ID (VLAN 10 = 10.1.10.0/24). Das Gateway ist die .1 im jeweiligen Subnetz.
- Im /24: .1 bis .19 statisch (Gateway, Geräte), .20 bis .254 DHCP. Das Management-VLAN ist statisch.
- Verbindungen zwischen Geräten sind /30-Netze aus 10.0.0.0/24, Loopbacks 10.255.0.x/32 (Router-IDs).
- Internet-Link 203.0.113.0/30 (Dokumentationsbereich, Platzhalter für öffentliche Adressen), NAT/PAT auf R0.
- Gateways der Office-A-VLANs sind auf die beiden Distribution-Switches verteilt: DSW-A1 für VLAN 10, 20, 99, DSW-A2 für VLAN 30, 40.
- DHCP und DNS zentral auf S-A2 (10.2.200.10), Office A nutzt DHCP-Relay (`ip helper-address`).

## VLANs und Subnetze

| VLAN | Name | Standort | Subnetz | Gateway | Gateway-Gerät | DHCP-Bereich | Bemerkung |
|---|---|---|---|---|---|---|---|
| 10 | HR | Office A | 10.1.10.0/24 | 10.1.10.1 | DSW-A1, SVI Vlan10 | 10.1.10.20 – .254 | PC1, PC2 |
| 20 | ACCOUNTING | Office A | 10.1.20.0/24 | 10.1.20.1 | DSW-A1, SVI Vlan20 | 10.1.20.20 – .254 | PC3 |
| 30 | LOGISTICS | Office A | 10.1.30.0/24 | 10.1.30.1 | DSW-A2, SVI Vlan30 | 10.1.30.20 – .254 | PC5, PC6 |
| 40 | VOICE | Office A | 10.1.40.0/24 | 10.1.40.1 | DSW-A2, SVI Vlan40 | 10.1.40.20 – .254 | IPPh-A1 (Voice-VLAN am Port ASW-A4 F0/2) |
| 99 | IT-MGMT | Office A | 10.1.99.0/24 | 10.1.99.1 | DSW-A1, SVI Vlan99 | statisch | Geräteverwaltung, Admin-PC4 |
| 50 | PUBLIC | Office B | 10.2.50.0/24 | 10.2.50.1 | R-B1, Subinterface G0/1.50 | 10.2.50.20 – .254 | L-B1 (WLAN über WLC/AP) |
| 99 | IT-MGMT | Office B | 10.2.99.0/24 | 10.2.99.1 | R-B1, Subinterface G0/1.99 | statisch, AP per DHCP (.50 – .59) | Geräteverwaltung, Admin-PC7, WLC-B1, AP |
| – | SERVER | Office B | 10.2.200.0/24 | 10.2.200.1 | R-B1, G0/2 (Routed Port) | statisch | S-A2, kein VLAN |
| 999 | NATIVE | alle Trunks | – | – | – | – | ungenutzt, nur Native-VLAN |

## Verbindungsnetze

| Netz | Zweck | Gerät A | Gerät B |
|---|---|---|---|
| 10.0.0.0/30 | R0 – R-B1 (WAN) | R0 G0/0: 10.0.0.1 | R-B1 G0/0: 10.0.0.2 |
| 10.0.0.4/30 | R0 – Core | R0 G0/1: 10.0.0.5 | CSW-A1 G0/1: 10.0.0.6 |
| 10.0.0.8/30 | Core – Distribution 1 | CSW-A1 F0/2: 10.0.0.9 | DSW-A1 F0/2: 10.0.0.10 |
| 10.0.0.12/30 | Core – Distribution 2 | CSW-A1 F0/5: 10.0.0.13 | DSW-A2 F0/5: 10.0.0.14 |
| 203.0.113.0/30 | Internet (ISP) | R0 G0/2: 203.0.113.2 | ISP: 203.0.113.1 |
| 10.255.0.0/24 | Loopbacks (Router-IDs) | R0 .1, R-B1 .2 | CSW-A1 .3, DSW-A1 .4, DSW-A2 .5 |

## Geräteadressen

### Office A

| Gerät | Interface | IP / Präfix | Netz / VLAN | Gateway | Bemerkung |
|---|---|---|---|---|---|
| R0 | G0/0 | 10.0.0.1/30 | WAN R0 – R-B1 | – |  |
| R0 | G0/1 | 10.0.0.5/30 | Link R0 – CSW-A1 | – |  |
| R0 | G0/2 | 203.0.113.2/30 | Internet | 203.0.113.1 | NAT/PAT outside |
| R0 | Loopback0 | 10.255.0.1/32 | Loopback | – | OSPF Router-ID |
| CSW-A1 | G0/1 | 10.0.0.6/30 | Link R0 – CSW-A1 | – | Routed Port |
| CSW-A1 | F0/2 | 10.0.0.9/30 | Link CSW-A1 – DSW-A1 | – | Routed Port |
| CSW-A1 | F0/5 | 10.0.0.13/30 | Link CSW-A1 – DSW-A2 | – | Routed Port |
| CSW-A1 | Loopback0 | 10.255.0.3/32 | Loopback | – | OSPF Router-ID, Verwaltung (keine VLANs) |
| DSW-A1 | F0/2 | 10.0.0.10/30 | Link CSW-A1 – DSW-A1 | – | Routed Port |
| DSW-A1 | Vlan10 | 10.1.10.1/24 | VLAN 10 HR | – | Gateway |
| DSW-A1 | Vlan20 | 10.1.20.1/24 | VLAN 20 ACCOUNTING | – | Gateway |
| DSW-A1 | Vlan99 | 10.1.99.1/24 | VLAN 99 IT-MGMT | – | Gateway und Verwaltungs-IP |
| DSW-A1 | Loopback0 | 10.255.0.4/32 | Loopback | – | OSPF Router-ID |
| DSW-A2 | F0/5 | 10.0.0.14/30 | Link CSW-A1 – DSW-A2 | – | Routed Port |
| DSW-A2 | Vlan30 | 10.1.30.1/24 | VLAN 30 LOGISTICS | – | Gateway |
| DSW-A2 | Vlan40 | 10.1.40.1/24 | VLAN 40 VOICE | – | Gateway |
| DSW-A2 | Vlan99 | 10.1.99.2/24 | VLAN 99 IT-MGMT | – | nur Verwaltungs-IP, kein Gateway der Hosts |
| DSW-A2 | Loopback0 | 10.255.0.5/32 | Loopback | – | OSPF Router-ID |
| ASW-A1 | Vlan99 | 10.1.99.11/24 | VLAN 99 IT-MGMT | 10.1.99.1 | Verwaltungs-IP |
| ASW-A2 | Vlan99 | 10.1.99.12/24 | VLAN 99 IT-MGMT | 10.1.99.1 | Verwaltungs-IP |
| ASW-A3 | Vlan99 | 10.1.99.13/24 | VLAN 99 IT-MGMT | 10.1.99.1 | Verwaltungs-IP |
| ASW-A4 | Vlan99 | 10.1.99.14/24 | VLAN 99 IT-MGMT | 10.1.99.1 | Verwaltungs-IP |
| PC4 | FastEthernet0 | 10.1.99.100/24 | VLAN 99 IT-MGMT | 10.1.99.1 | Admin-PC (statisch) |
| PC1 | FastEthernet0 | DHCP | VLAN 10 HR | per DHCP |  |
| PC2 | FastEthernet0 | DHCP | VLAN 10 HR | per DHCP |  |
| PC3 | FastEthernet0 | DHCP | VLAN 20 ACCOUNTING | per DHCP |  |
| PC5 | FastEthernet0 | DHCP | VLAN 30 LOGISTICS | per DHCP |  |
| PC6 | FastEthernet0 | DHCP | VLAN 30 LOGISTICS | per DHCP |  |
| IPPh-A1 | Switch-Port | DHCP | VLAN 40 VOICE | per DHCP | Voice-VLAN |

### Office B

| Gerät | Interface | IP / Präfix | Netz / VLAN | Gateway | Bemerkung |
|---|---|---|---|---|---|
| R-B1 | G0/0 | 10.0.0.2/30 | WAN R0 – R-B1 | – |  |
| R-B1 | G0/1 | – | Trunk zu ASW-B1 | – | Router-on-a-Stick, ohne IP |
| R-B1 | G0/1.50 | 10.2.50.1/24 | VLAN 50 PUBLIC | – | Gateway |
| R-B1 | G0/1.99 | 10.2.99.1/24 | VLAN 99 IT-MGMT | – | Gateway |
| R-B1 | G0/1.999 | – | Native VLAN 999 | – | keine IP |
| R-B1 | G0/2 | 10.2.200.1/24 | Server-Netz | – | Gateway |
| R-B1 | Loopback0 | 10.255.0.2/32 | Loopback | – | OSPF Router-ID |
| ASW-B1 | Vlan99 | 10.2.99.11/24 | VLAN 99 IT-MGMT | 10.2.99.1 | Verwaltungs-IP |
| ASW-B2 | Vlan99 | 10.2.99.12/24 | VLAN 99 IT-MGMT | 10.2.99.1 | Verwaltungs-IP |
| WLC-B1 | Management | 10.2.99.20/24 | VLAN 99 IT-MGMT | 10.2.99.1 | Wireless LAN Controller |
| LWAP-B1 | G0/0 | DHCP (10.2.99.50 – .59) | VLAN 99 IT-MGMT | 10.2.99.1 |  |
| S-B1 | FastEthernet0 | 10.2.200.10/24 | Server-Netz | 10.2.200.1 | DHCP, DNS |
| PC7 | FastEthernet0 | 10.2.99.100/24 | VLAN 99 IT-MGMT | 10.2.99.1 | Admin-PC (statisch) |
| L-B1 | Wireless0 | DHCP | VLAN 50 PUBLIC | per DHCP | WLAN über AP/WLC |
