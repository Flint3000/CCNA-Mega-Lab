# CCNA-Lab: Campus-Netzwerk mit Hauptstandort und Branch Office

Zwei-Standort-Netzwerk in Cisco Packet Tracer, gebaut als praktische Vorbereitung auf die **CCNA 200-301 (v1.1)** und als dokumentiertes Portfolio-Projekt. Die Topologie ist an ein Lab aus Jeremys CCNA-Kurs angelehnt. Konfiguration und Dokumentation sind von mir.

## Topologie

![Topologie](docs/topology.png)

| Standort | Beschreibung |
|---|---|
| **Office A** (Hauptstandort) | Drei-Stufen-Design: 4 Access-Switches (ASW-A1 bis ASW-A4), 2 Multilayer-Distribution-Switches (DSW-A1, DSW-A2) als VLAN-Gateways, Core-Switch CSW-A1, Router R0 mit Anbindung zum Internet |
| **Office B** (Branch) | Router R-B1 (Router-on-a-Stick), 2 Access-Switches (ASW-B1, ASW-B2), WLC-B1 mit Access Point LWAP-B1, Server S-B1 (DHCP/DNS) |

**Namensschema:** Rolle, Standort, Nummer. `ASW` = Access, `DSW` = Distribution, `CSW` = Core, `R` = Router (z. B. `ASW-A1`, `DSW-A2`, `R-B1`).

## CCNA-Themen im Lab

| Bereich | Themen | Stand |
|---|---|---|
| Network Access | VLANs, Voice-VLAN, Trunks, Native-VLAN, EtherChannel (LACP), Rapid PVST+, PortFast, BPDU Guard, Root Guard, CDP | umgesetzt |
| IP Connectivity | Inter-VLAN-Routing (SVI und Router-on-a-Stick), statische Default-Route, OSPFv2 (Single Area) | umgesetzt |
| IP Services | DHCP mit Relay, DNS, NAT/PAT, NTP, Syslog, SSH | geplant |
| Security | ACLs, Port Security, DHCP Snooping, DAI | geplant |
| Wireless und IPv6 | WLAN mit WLC, optional IPv6 | geplant |

## Aufbau in Phasen

| Phase | Inhalt | Status |
|---|---|---|
| [01](phases/01-basis.md) | Basiskonfiguration aller Geräte | ☑ |
| [02](phases/02-vlans-trunks.md) | Layer 2: VLANs, Trunks, EtherChannel, Spanning Tree | ☑ |
| [03](phases/03-ip-connectivity.md) | IP Connectivity: Adressierung, Routing, OSPF | ☑ |
| 04 | IP Services: DHCP, DNS, NAT, NTP, Syslog, SSH | geplant |
| 05 | Security: ACLs, Port Security, DHCP Snooping, DAI | geplant |
| 06 | Wireless und optional IPv6 | geplant |

Jede Phase hat dieselbe kurze Struktur: Ziel, CCNA-Bezug, Entscheidungen, Konfiguration, Verifikation, Probleme und Learnings (Vorlage: [`phases/00-vorlage.md`](phases/00-vorlage.md)).

## Dokumentation

- [`docs/ip-plan.md`](docs/ip-plan.md): Adressierung, VLANs, Subnetze
- [`docs/port-table.md`](docs/port-table.md): Verkabelung, Port für Port

## Adressierung auf einen Blick

| VLAN | Name | Subnetz | Standort |
|---|---|---|---|
| 10 | HR | 10.1.10.0/24 | A |
| 20 | ACCOUNTING | 10.1.20.0/24 | A |
| 30 | LOGISTICS | 10.1.30.0/24 | A |
| 40 | VOICE | 10.1.40.0/24 | A |
| 50 | PUBLIC | 10.2.50.0/24 | B |
| 99 | IT-MGMT | 10.1.99.0/24 und 10.2.99.0/24 | A und B |

## Bewusste Vereinfachungen

- Die VLAN-Gateways in Office A sind auf DSW-A1 (VLAN 10, 20, 99) und DSW-A2 (VLAN 30, 40) verteilt, ohne FHRP. Fällt ein DSW aus, haben seine VLANs kein Gateway mehr.
- Office B hat nur einen Weg zum Router: Fällt ASW-B1 aus, ist ASW-B2 mit AP und WLC vom Router getrennt.
- Loop Guard ist in Packet Tracer nicht verfügbar und deshalb nicht konfiguriert.
- Der Internetzugang wird durch einen direkt an R0 angeschlossenen ISP-Router simuliert.
- DHCP, DNS und Syslog laufen zentral auf einem Server (S-B1) in Office B. Fällt der WAN-Link aus, erreicht Office A den Server nicht mehr und bekommt keine neuen Adressen.
- NTP läuft nur auf Routern und Switches, ohne Authentifizierung. R0 ist `ntp master` mit seiner eigenen Uhr als Quelle. WLC, Access Point und IP-Phone sind ausgenommen.

## Hinweise

- Alle Passwörter in diesem Repository sind Lab-Passwörter und durch Platzhalter ersetzt.
- Das Lab ist eine Simulation. Einzelne Befehle und Funktionen verhalten sich in Packet Tracer anders als auf echter Hardware.
