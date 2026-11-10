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
- `service password-encryption` zusätzlich als Lernbeispiel: Es verschleiert Klartext-Passwörter nur als Typ 7, und das ist umkehrbar. Da hier konsequent `secret` verwendet wird, gibt es kaum Klartext-Passwörter, der Befehl bringt also wenig. Der Vergleich Typ 7 gegen Typ 5 gehört in die Learnings.
- Lokale Benutzer (`username ... secret`) statt einer gemeinsamen Line-Passwort-Lösung, damit Zugriffe einem Konto zuzuordnen sind.
- Einheitliches Namensschema nach Rolle, Standort und Nummer: `ASW` (Access), `DSW` (Distribution), `CSW` (Core), `R` (Router), z. B. `ASW-A1`, `DSW-A2`, `R-B1`.

## Konfiguration
Vorlage: [`../configs/00-basis-vorlage.txt`](../configs/00-basis-vorlage.txt). Pro Gerät `<HOSTNAME>`, `<ADMIN-USER>` und `<LAB-PASSWORT>` ersetzen.

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
| `show running-config \| include hostname` | alle | Hostname stimmt |
| `show running-config \| section line` | alle | `login local` und Timeout gesetzt |
| `show cdp neighbors` | alle | Nachbarn entsprechen der Port-Tabelle |
| `show ip interface brief` | Router | Interfaces und Status |
| `show interfaces status` | Switches | Ports `connected` |

## Probleme und Learnings
_Hier eintragen._

## Offene Punkte
SSH (Phase 04). Management-IPs sind in Phase 02 umgesetzt.
