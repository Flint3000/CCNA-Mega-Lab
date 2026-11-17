# Portbelegung

Alle belegten Ports pro Gerät mit Gegenstelle. Jede Verbindung steht auf beiden Seiten. Auf allen Trunks gilt Native-VLAN 999 und `switchport nonegotiate`. Alle PCs sind an FastEthernet0 (F0) angeschlossen. Ungenutzte Ports sind abgeschaltet (`shutdown`).

## Office A (Hauptstandort)

| Gerät | Port | Verbunden mit | Gegenstelle | Modus | VLAN(s) | Bemerkung |
|---|---|---|---|---|---|---|
| R0 | G0/0 | R-B1 | G0/0 | Routed | – | WAN Office A – Office B, Link-Netz 10.0.0.0/30 |
| R0 | G0/1 | CSW-A1 | G0/1 | Routed | – | Link-Netz 10.0.0.4/30 |
| R0 | G0/2 | Internet | – | Routed | – | Link-Netz 203.0.113.0/30 |
| CSW-A1 | F0/2 | DSW-A1 | F0/2 | Routed | – | Link-Netz 10.0.0.8/30 |
| CSW-A1 | F0/5 | DSW-A2 | F0/5 | Routed | – | Link-Netz 10.0.0.12/30 |
| CSW-A1 | G0/1 | R0 | G0/1 | Routed | – | Link-Netz 10.0.0.4/30 |
| DSW-A1 | F0/1 | ASW-A2 | F0/2 | Trunk | 20, 99 |  |
| DSW-A1 | F0/2 | CSW-A1 | F0/2 | Routed | – | Link-Netz 10.0.0.8/30 |
| DSW-A1 | F0/3 | ASW-A1 | F0/3 | Trunk | 10, 99 |  |
| DSW-A1 | F0/4 | ASW-A3 | F0/4 | Trunk | 99 |  |
| DSW-A1 | F0/5 | ASW-A4 | F0/5 | Trunk | 30, 40, 99 |  |
| DSW-A1 | F0/7 | DSW-A2 | F0/7 | EtherChannel | 10, 20, 30, 40, 99 | Po1, LACP |
| DSW-A1 | F0/8 | DSW-A2 | F0/8 | EtherChannel | 10, 20, 30, 40, 99 | Po1, LACP |
| DSW-A2 | F0/1 | ASW-A3 | F0/2 | Trunk | 99 |  |
| DSW-A2 | F0/2 | ASW-A4 | F0/3 | Trunk | 30, 40, 99 |  |
| DSW-A2 | F0/3 | ASW-A2 | F0/1 | Trunk | 20, 99 |  |
| DSW-A2 | F0/4 | ASW-A1 | F0/4 | Trunk | 10, 99 |  |
| DSW-A2 | F0/5 | CSW-A1 | F0/5 | Routed | – | Link-Netz 10.0.0.12/30 |
| DSW-A2 | F0/7 | DSW-A1 | F0/7 | EtherChannel | 10, 20, 30, 40, 99 | Po1, LACP |
| DSW-A2 | F0/8 | DSW-A1 | F0/8 | EtherChannel | 10, 20, 30, 40, 99 | Po1, LACP |
| ASW-A1 | F0/1 | PC1 | F0 | Access | 10 |  |
| ASW-A1 | F0/2 | PC2 | F0 | Access | 10 |  |
| ASW-A1 | F0/3 | DSW-A1 | F0/3 | Trunk | 10, 99 |  |
| ASW-A1 | F0/4 | DSW-A2 | F0/4 | Trunk | 10, 99 |  |
| ASW-A2 | F0/1 | DSW-A2 | F0/3 | Trunk | 20, 99 |  |
| ASW-A2 | F0/2 | DSW-A1 | F0/1 | Trunk | 20, 99 |  |
| ASW-A2 | F0/3 | PC3 | F0 | Access | 20 |  |
| ASW-A3 | F0/1 | PC4 | F0 | Access | 99 | Admin-PC |
| ASW-A3 | F0/2 | DSW-A2 | F0/1 | Trunk | 99 |  |
| ASW-A3 | F0/4 | DSW-A1 | F0/4 | Trunk | 99 |  |
| ASW-A4 | F0/1 | PC5 | F0 | Access | 30 |  |
| ASW-A4 | F0/2 | IPPh-A1 | Switch-Port | Access | 30 (Access), 40 (Voice) | Telefon-Port |
| ASW-A4 | F0/3 | DSW-A2 | F0/2 | Trunk | 30, 40, 99 |  |
| ASW-A4 | F0/5 | DSW-A1 | F0/5 | Trunk | 30, 40, 99 |  |
| IPPh-A1 | PC-Port | PC6 | F0 | – | 30 | PC hinter dem Telefon |
| IPPh-A1 | Switch-Port | ASW-A4 | F0/2 | – | 30 (Access), 40 (Voice) | Telefon-Port |
| PC1 | F0 | ASW-A1 | F0/1 | – | 10 |  |
| PC2 | F0 | ASW-A1 | F0/2 | – | 10 |  |
| PC3 | F0 | ASW-A2 | F0/3 | – | 20 |  |
| PC4 | F0 | ASW-A3 | F0/1 | – | 99 | Admin-PC |
| PC5 | F0 | ASW-A4 | F0/1 | – | 30 |  |
| PC6 | F0 | IPPh-A1 | PC-Port | – | 30 | PC hinter dem Telefon |

## Office B (Branch)

| Gerät | Port | Verbunden mit | Gegenstelle | Modus | VLAN(s) | Bemerkung |
|---|---|---|---|---|---|---|
| R-B1 | G0/0 | R0 | G0/0 | Routed | – | WAN Office A – Office B, Link-Netz 10.0.0.0/30 |
| R-B1 | G0/1 | ASW-B1 | G0/1 | Trunk | 50, 99 | Router-on-a-Stick |
| R-B1 | G0/2 | S-A2 | FastEthernet0 | Routed | – | Server-Netz 10.2.200.0/24 |
| ASW-B1 | F0/1 | PC7 | F0 | Access | 99 | Admin-PC |
| ASW-B1 | F0/2 | ASW-B2 | F0/1 | Trunk | 50, 99 |  |
| ASW-B1 | G0/1 | R-B1 | G0/1 | Trunk | 50, 99 | Router-on-a-Stick |
| ASW-B2 | F0/1 | ASW-B1 | F0/2 | Trunk | 50, 99 |  |
| ASW-B2 | G0/1 | WLC-B1 | G0 | Trunk | 50, 99 |  |
| ASW-B2 | G0/2 | LWAP-B1 | G0/0 | Access | 99 | Access Point |
| LWAP-B1 | G0/0 | ASW-B2 | G0/2 | – | 99 | Access Point |
| WLC-B1 | G0 | ASW-B2 | G0/1 | – | 50, 99 |  |
| PC7 | F0 | ASW-B1 | F0/1 | – | 99 | Admin-PC |
| S-A2 | FastEthernet0 | R-B1 | G0/2 | – | – | Server-Netz 10.2.200.0/24 |
