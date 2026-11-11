# Phase 01: Basiskonfiguration aller Geräte

## Ziel
Alle Router und Switches haben einen eindeutigen Hostnamen, abgesicherten Zugang über die Konsole und eine einheitliche Grundkonfiguration.

## CCNA-Bezug
- 5.3 Device Access Control mit lokalen Passwörtern
- 2.8 Management-Zugang (Konsole, später SSH)
- 2.3 Layer-2-Discovery (CDP/LLDP) zur Kontrolle der Verkabelung

## Ausgangslage
Topologie ist in Packet Tracer verkabelt, Geräte sind unkonfiguriert. Verkabelung laut [`../docs/port-table.md`](../docs/port-table.md).

## Entscheidungen
- `enable secret` statt `enable password`: Das Secret wird gehasht gespeichert, ein Enable-Passwort nur schwach (Typ 7) verschleiert.
- **`service password-encryption` wird nicht verwendet:** Es verschleiert nur Klartext-Passwörter als Typ 7 (umkehrbar). Da durchgehend `secret` verwendet wird, die schon gehasht gespeichert sind, bringt der Befehl nichts.
- Lokale Benutzer (`username ... secret`) statt einer gemeinsamen Line-Passwort-Lösung, damit Zugriffe einem Konto zuzuordnen sind.
- Einheitliches Namensschema nach Rolle, Standort und Nummer: `ASW` (Access), `DSW` (Distribution), `CSW` (Core), `R` (Router), z. B. `ASW-A1`, `DSW-A2`, `R-B1`.
- **Hash-Typ:** `algorithm-type` steht in Packet Tracer nicht zur Verfügung, deshalb Typ 5. Auf echter Hardware würde ich Typ 8 oder 9 nehmen (`algorithm-type scrypt`).

## Konfiguration
Basis-Config für jedes Gerät (`<HOSTNAME>`, `<ADMIN-USER>` und `<LAB-PASSWORT>` pro Gerät ersetzen):
```
enable
configure terminal
hostname <HOSTNAME>
enable secret <LAB-PASSWORT>
username <ADMIN-USER> secret <LAB-PASSWORT>
line console 0
 login local
end
copy running-config startup-config
```

| Gerät | Hostname |
|---|---|
| Router (Internet-Anbindung) | R0 |
| Router Office B | R-B1 |
| Core-Switch | CSW-A1 |
| Distribution-Switches | DSW-A1, DSW-A2 |
| Access-Switches Office A | ASW-A1, ASW-A2, ASW-A3, ASW-A4 |
| Access-Switches Office B | ASW-B1, ASW-B2 |

## Verifikation

| Befehl | Gerät | Erwartung |
|---|---|---|
| `show cdp neighbors` | DSW-A1 | Nachbarn entsprechen der Port-Tabelle |
| `show running-config \| section line` | DSW-A2 | `login local` an der Konsole |
| `show running-config \| include secret` | ASW-A2 | `enable secret` und Benutzer als Hash (geschwärzt) |

#### Nachbarn: `show cdp neighbors`

DSW-A1 (DSW-A2 erscheint dreimal, weil die Verbindung über Po1 mit den Mitgliedsports F0/7 und F0/8 läuft):
```
DSW-A1#show cdp neighbors
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone
Device ID    Local Intrfce   Holdtme    Capability   Platform    Port ID
ASW-A1       Fas 0/3          136            S       2960        Fas 0/3
ASW-A3       Fas 0/4          130            S       2960        Fas 0/4
CSW-A1       Fas 0/2          149                    3560        Fas 0/2
ASW-A4       Fas 0/5          168            S       2960        Fas 0/5
ASW-A2       Fas 0/1          130            S       2960        Fas 0/2
DSW-A2       Por 1            149                    3560        Fas 0/7
DSW-A2       Por 1            149                    3560        Fas 0/8
DSW-A2       Por 1            149                    3560        Por 1
```

#### Konsole: `show running-config | section line`

DSW-A2 (`login local` an der Konsole, die VTY-Leitungen folgen mit SSH in Phase 04):
```
DSW-A2#show run | section line
line con 0
 login local
line aux 0
line vty 0 4
 login
```

#### Secrets: `show running-config | include secret`

ASW-A2 (Hashes geschwärzt):
```
ASW-A2#show run | include secret
enable secret 5 <REDACTED>
username ccna secret 5 <REDACTED>
```

## Probleme und Learnings
- **Hash-Typen sind keine Rangfolge der Stärke.** Typ 7 (`service password-encryption`) ist eine umkehrbare Verschleierung, Typ 5 ein schwacher MD5-Hash, Typ 8 und 9 sind stark. Secrets sind schon gehasht, deshalb bringt `service password-encryption` bei konsequenter Nutzung von `secret` nichts.
