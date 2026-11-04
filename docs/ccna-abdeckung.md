# CCNA-200-301-v1.1-Abdeckung

Zuordnung der Prüfungsthemen (Blueprint v1.1) zum Lab. Die Nummern folgen der offiziellen Themenliste von Cisco (Learning Network). Status: ☐ offen · ☑ umgesetzt · – nur Theorie.

## 1.0 Network Fundamentals

| Nr. | Thema | Im Lab | Phase | Status |
|---|---|---|---|---|
| 1.1 | Rolle von Routern, Switches, APs, Controllern, Endgeräten, Servern | Topologie mit allen Gerätetypen | – | ☐ |
| 1.2 | Topologie-Architekturen (Two-Tier, WAN, SOHO) | Campus mit Distribution, Branch, WAN | – | ☐ |
| 1.4 | Interface- und Kabelprobleme erkennen | `show interfaces`, Duplex/Speed | 01 | ☐ |
| 1.6 / 1.7 | IPv4-Adressierung, Subnetting, private Adressen | IP-Plan | 05 | ☐ |
| 1.8 / 1.9 | IPv6-Adressierung und Adresstypen | optional | 10 | ☐ |
| 1.10 | IP-Parameter auf Clients prüfen | `ipconfig` auf den PCs | 07 | ☐ |
| 1.11 | Wireless-Grundlagen (SSID, Kanäle, Verschlüsselung) | WLAN mit WLC | 09 | ☐ |
| 1.13 | Switching-Konzepte (MAC-Tabelle) | `show mac address-table` | 02 | ☐ |

## 2.0 Network Access

| Nr. | Thema | Im Lab | Phase | Status |
|---|---|---|---|---|
| 2.1 | VLANs, Access-Ports, Voice-VLAN, Inter-VLAN | VLAN 10/20/30/40/50/99 | 02, 05 | ☐ |
| 2.2 | Trunks, 802.1Q, Native-VLAN | Trunks, Native-VLAN 999 | 02 | ☐ |
| 2.3 | CDP und LLDP | Nachbarschaftskontrolle | 01 | ☐ |
| 2.4 | EtherChannel (LACP) | SW1 und SW6 | 03 | ☐ |
| 2.5 | Rapid PVST+, PortFast, Root Guard, Loop Guard, BPDU Guard/Filter | Root-Bridge-Planung, Access-Ports | 03 | ☐ |
| 2.6 / 2.7 | Wireless-Architekturen, WLC-Verbindungen | WLC und Lightweight AP | 09 | ☐ |
| 2.8 | Management-Zugang (Konsole, SSH, HTTP/S, AAA) | Konsole, SSH | 01, 04 | ☐ |
| 2.9 | WLAN-GUI-Konfiguration | WLC-Oberfläche | 09 | ☐ |

## 3.0 IP Connectivity

| Nr. | Thema | Im Lab | Phase | Status |
|---|---|---|---|---|
| 3.1 / 3.2 | Routing-Tabelle, Weiterleitungsentscheidung | `show ip route` | 06 | ☐ |
| 3.3 | Statisches Routing (Default, Netz, Host, Floating) | Default-Route R0, weitere nach Bedarf | 06 | ☐ |
| 3.4 | OSPFv2 Single Area | R0, R1, MLS0 | 06 | ☐ |
| 3.5 | First-Hop-Redundancy-Protokolle (nur Konzept) | nicht im Lab | – | – |

## 4.0 IP Services

| Nr. | Thema | Im Lab | Phase | Status |
|---|---|---|---|---|
| 4.1 | Inside Source NAT (statisch und Pool/PAT) | Router0 | 07 | ☐ |
| 4.2 | NTP (Client und Server) | Router0 oder Server0 als Quelle | 07 | ☐ |
| 4.3 | DHCP und DNS | Server0 | 07 | ☐ |
| 4.4 / 4.5 | SNMP-Funktion, Syslog-Level | Syslog zu Server0 | 07 | ☐ |
| 4.6 | DHCP Client und Relay | `ip helper-address` | 07 | ☐ |
| 4.7 | QoS (nur Konzept) | nicht im Lab | – | – |
| 4.8 | SSH | alle Geräte | 04 | ☐ |
| 4.9 | TFTP/FTP | Config-Backup auf Server0 (optional) | 07 | ☐ |

## 5.0 Security Fundamentals

| Nr. | Thema | Im Lab | Phase | Status |
|---|---|---|---|---|
| 5.1 / 5.2 | Sicherheitskonzepte und -programme (Theorie) | – | – | – |
| 5.3 / 5.4 | Zugriffskontrolle mit lokalen Passwörtern, Passwort-Richtlinien | Basiskonfiguration | 01 | ☐ |
| 5.5 | IPsec-VPNs (nur Konzept) | nicht im Lab | – | – |
| 5.6 | ACLs | SSH nur aus IT-MGMT, Public isolieren | 08 | ☐ |
| 5.7 | DHCP Snooping, DAI, Port Security | Access-Ports | 08 | ☐ |
| 5.8 | AAA (nur Konzept) | nicht im Lab | – | – |
| 5.9 / 5.10 | WPA/WPA2/WPA3, WPA2-PSK in der GUI | WLAN Public | 09 | ☐ |

## 6.0 Automation and Programmability

Reine Theorie (Automatisierung, Controller-Netze, KI/ML, REST-APIs, Ansible/Terraform, JSON). Nicht im Lab, aber prüfungsrelevant.
