# Design-Entscheidungen

Kurze Begründungen, warum das Netz so aufgebaut ist. Je Entscheidung: Ausgangsfrage, Entscheidung, Alternative, Konsequenz.

## Vorlage

### <Titel>
- **Ausgangsfrage:**
- **Entscheidung:**
- **Alternative(n):**
- **Konsequenz / Einschränkung:**

## Einträge

### Branch ohne Gateway-Redundanz
- **Ausgangsfrage:** Soll der Branch zwei Gateway-Pfade haben?
- **Entscheidung:** Router-on-a-Stick über einen Trunk (R1 ↔ SW3), SW7 hängt über SW3 am Router.
- **Alternative(n):** Zweiter Router plus FHRP.
- **Konsequenz:** Fällt SW3 aus, ist SW7 (AP, WLC) vom Router getrennt. Für ein kleines Branch Office bewusst akzeptiert.

### Management-VLAN 99 in beiden Standorten
- **Ausgangsfrage:** Reicht ein Management-VLAN für beide Standorte?
- **Entscheidung:** Gleiche VLAN-ID, aber getrennte Subnetze, da die Standorte geroutet verbunden sind.
- **Konsequenz:** SSH auf die Geräte wird per ACL nur aus den 99er-Netzen erlaubt.

### Native-VLAN 999
- **Entscheidung:** Ungenutztes Native-VLAN auf allen Trunks, damit Management- und Benutzerverkehr nicht ungetaggt über Trunks läuft.
