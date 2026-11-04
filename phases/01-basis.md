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
- `enable secret` statt `enable password`: Das Secret wird gehasht gespeichert, ein Enable-Passwort nur schwach (Typ 7) verschlüsselt
- `service password-encryption` zusätzlich, weil sonst auch Line-Passwörter im Klartext in der Config stehen. Typ 7 ist umkehrbar und kein echter Schutz.
- Lokale Benutzer (`username ... secret`) statt einer gemeinsamen Line-Passwort-Lösung, damit Zugriffe einem Konto zuzuordnen sind.
- Einheitliche Hostnamen wie in der Topologie (Router0, Router1, MLS0, SW0 bis SW7).

## Konfiguration
Vorlage: [`../configs/00-basis-vorlage.txt`](../configs/00-basis-vorlage.txt). Pro Gerät `<HOSTNAME>`, `<ADMIN-USER>` und `<LAB-PASSWORT>` ersetzen.

| Gerät | Hostname |
|---|---|
| Router0 | Router0 |
| Router1 | Router1 |
| Multilayer SW0 | MLS0 |
| Switch0 bis Switch7 | SW0 bis SW7 |

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
SSH (Phase 04), Management-IPs (Phase 04/05).
