# Phase 04: IP Services

## Ziel
Adressvergabe, Namensauflösung, Internetzugang, Zeitsynchronisation, Logging und sicherer Zugriff funktionieren.

## CCNA-Bezug
4.1 NAT, 4.2 NTP, 4.3 DHCP und DNS, 4.4 und 4.5 SNMP und Syslog, 4.6 DHCP-Relay, 4.8 SSH

## Entscheidungen
- **DHCP und DNS:** Ein zentraler Server S-B1 in Office B, Office A nutzt DHCP-Relay über den WAN-Link. Ein Ausfall des WAN-Links macht den Server für Office A unerreichbar (bewusste Vereinfachung).
- **NAT:** Alle drei Arten einmal im Lab, damit die CCNA-Themen abgedeckt sind. Der ISP-Link ist dafür ein /28 (203.0.113.0/28).
  - Office A mit Pool (.4 bis .14, elf Adressen für die Geräte in Office A mit Reserve; gleichzeitige Geräte sind durch die Poolgröße begrenzt).
  - Office B mit PAT über die Adresse von R0 G0/2 (.2), weil im öffentlichen WLAN viele Clients zu erwarten sind.
  - S-B1 mit statischem NAT auf .3, nur als Übung. In der Praxis macht man einen internen Infrastruktur-Server nicht aus dem Internet erreichbar.
- **NTP:** R0 ist die Zeitquelle (`ntp master`) für alle Router und Switches in beiden Offices. WLC, Access Point und IP-Phone sind ausgenommen, NTP-Authentifizierung entfällt.
- **Syslog:** Alle Geräte loggen ab Level 5 (Notifications) an S-B1.
- **SSH:** Auf allen Routern und Switches (Version 2, RSA 2048), damit das IT-MGMT-Netz alle Geräte verwalten kann.
- **Config-Backup:** per TFTP auf S-B1.

## Konfiguration

_Wesentliche Auszüge, ein Gerät pro Konfigurationstyp. Noch offen: Syslog, TFTP-Backup._

### NTP

R0 (Zeitquelle):
```
clock timezone CET 1
service timestamps log datetime msec
ntp master
ntp update-calendar
!
R0# clock set <hh:mm:ss UTC> 18 November 2026
```

Alle anderen Router und Switches:
```
clock timezone CET 1
service timestamps log datetime msec
ntp server 10.255.0.1
ntp update-calendar        ! nicht auf den ASW verfügbar
```

### SSH

Auf allen Routern und Switches (VTY `0 4`, auf den ASW `0 15`):
```
ip domain-name cisco.lab
crypto key generate rsa modulus 2048
ip ssh version 2
!
line vty 0 4
 exec-timeout 10
 transport input ssh
 login local
```

### DHCP-Relay

DSW-A1 (VLAN 10, 20), DSW-A2 (VLAN 30, 40) und R-B1 (Subinterfaces 50, 99):
```
interface vlan 10
 ip helper-address 10.2.200.10
!
interface g0/1.50
 ip helper-address 10.2.200.10
```

VLAN 99 in Office A hat kein Relay (statische Adressen). In Office B braucht G0/1.99 es für den AP.

### DHCP-Server S-B1

Konfiguration über Services, DHCP (Packet-Tracer-Server ohne CLI). DNS-Server in allen Pools: 10.2.200.10.

| Pool | Gateway | Bereich | Max. Nutzer |
|---|---|---|---|
| VLAN 10 | 10.1.10.1 | 10.1.10.20 – .254 | 235 |
| VLAN 20 | 10.1.20.1 | 10.1.20.20 – .254 | 235 |
| VLAN 30 | 10.1.30.1 | 10.1.30.20 – .254 | 235 |
| VLAN 40 | 10.1.40.1 | 10.1.40.20 – .254 | 235 |
| VLAN 50 | 10.2.50.1 | 10.2.50.20 – .254 | 235 |
| VLAN 99 (nur AP) | 10.2.99.1 | 10.2.99.50 – .59 | 10 |

![DHCP-Pools auf S-B1](../docs/dhcp-pools.png)

### DNS

Server S-B1 (Services, DNS): A-Records für die Infrastruktur. Router und Layer-3-Switches zeigen auf ihre Loopback, die ASW auf ihre VLAN-99-Adresse, dazu S-B1, PC4 und PC7. PCs mit DHCP-Adressen sind nicht eingetragen, weil sich ihre Adressen ändern können.

![DNS-Records auf S-B1](../docs/dns-records.png)

Auf R0, CSW-A1, DSW-A1, DSW-A2 und R-B1:
```
ip name-server 10.2.200.10
```

PC4 und PC7 (statische Adressen) haben 10.2.200.10 manuell als DNS-Server eingetragen.

### NAT

R0 (G0/0 und G0/1 inside, G0/2 outside; Pool .4 bis .14 für Office A, PAT für Office B, statisches NAT für S-B1 nur als Übung):
```
interface g0/0
 ip nat inside
interface g0/1
 ip nat inside
interface g0/2
 ip nat outside
!
access-list 1 permit 10.2.0.0 0.0.255.255
access-list 2 permit 10.1.0.0 0.0.255.255
!
ip nat pool OfficeA 203.0.113.4 203.0.113.14 netmask 255.255.255.240
ip nat inside source list 1 interface g0/2 overload
ip nat inside source list 2 pool OfficeA
ip nat inside source static 10.2.200.10 203.0.113.3
```

## Verifikation

| Befehl | Gerät | Erwartung |
|---|---|---|
| `show ntp status`, `show clock detail` | R0, ASW-A1 | synchronisiert |
| `ipconfig` | PC1, PC5, PC6, L-B1 | Adresse, Gateway und DNS aus dem richtigen Pool |
| `ping R-B1` | DSW-A1 | Name wird über den DNS-Server aufgelöst |
| `show ip nat translations` | R0 | Übersetzungen nach einem Ping ins Internet |
| `show logging` | ein Router | Meldungen, Logging-Host S-B1 |
| `ssh -l <User> 10.255.0.1` und `10.1.99.14` | PC4 und PC7 | Anmeldung per SSH möglich |

### NTP

R0:
```
R0# show clock detail
10:36:46.359 cet Wed Nov 18 2026
Time source is NTP

R0# show ntp status
Clock is synchronized, stratum 8, reference is 127.127.1.1
reference time is FFFFFFFFEE79CB89.000003BD (9:36:9.957 UTC Wed Nov 18 2026)
clock offset is 0.00 msec, root delay is 0.00 msec
```

ASW-A1:
```
ASW-A1# show ntp status
Clock is synchronized, stratum 9, reference is 10.0.0.5
reference time is FFFFFFFFEE79CB3B.00000399 (9:34:51.921 UTC Wed Nov 18 2026)
clock offset is 0.00 msec, root delay is 0.00 msec

ASW-A1# show clock detail
10:34:59.687 CET Wed Nov 18 2026
Time source is NTP
```

### DHCP

PC1 (VLAN 10):
```
C:\>ipconfig /renew
   IP Address......................: 10.1.10.20
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: 10.1.10.1
   DNS Server......................: 10.2.200.10
```

PC5 (VLAN 30):
```
C:\>ipconfig /renew
   IP Address......................: 10.1.30.20
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: 10.1.30.1
   DNS Server......................: 10.2.200.10
```

PC6 (VLAN 30, hinter dem IP-Phone):
```
C:\>ipconfig /renew
   IP Address......................: 10.1.30.21
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: 10.1.30.1
   DNS Server......................: 10.2.200.10
```

_L-B1: folgt._

### DNS

DSW-A1:
```
DSW-A1# ping R-B1
Translating "R-B1"...domain server (10.2.200.10)
Sending 5, 100-byte ICMP Echos to 10.255.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

DSW-A1# show hosts
Default Domain is cisco.lab
Name/address lookup uses domain service
Name servers are 10.2.200.10

Host                      Port  Flags      Age Type   Address(es)
asw-a1                    None  (temp, OK)  0   IP      10.1.99.11
asw-a2                    None  (temp, OK)  0   IP      10.1.99.12
asw-a3                    None  (temp, OK)  0   IP      10.1.99.13
asw-a4                    None  (temp, OK)  0   IP      10.1.99.14
asw-b1                    None  (temp, OK)  0   IP      10.2.99.11
asw-b2                    None  (temp, OK)  0   IP      10.2.99.12
csw-a1                    None  (temp, OK)  0   IP      10.255.0.3
dsw-a1                    None  (temp, OK)  0   IP      10.255.0.4
dsw-a2                    None  (temp, OK)  0   IP      10.255.0.5
pc4                       None  (temp, OK)  0   IP      10.1.99.100
pc7                       None  (temp, OK)  0   IP      10.2.99.100
r-b1                      None  (temp, OK)  0   IP      10.255.0.2
r0                        None  (temp, OK)  0   IP      10.255.0.1
s-b1                      None  (temp, OK)  0   IP      10.2.200.10
```

### NAT

Konfiguration auf R0:
```
R0# show run | include ip nat
 ip nat inside
 ip nat inside
 ip nat outside
ip nat pool OfficeA 203.0.113.4 203.0.113.14 netmask 255.255.255.240
ip nat inside source list 1 interface GigabitEthernet0/2 overload
ip nat inside source list 2 pool OfficeA
ip nat inside source static 10.2.200.10 203.0.113.3
```

Pings von PC1 und PC5 (Office A, Pool) und PC7 (Office B, PAT) auf den ISP (203.0.113.1):
```
R0# show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 203.0.113.2:10    10.2.99.100:10     203.0.113.1:10     203.0.113.1:10
icmp 203.0.113.2:11    10.2.99.100:11     203.0.113.1:11     203.0.113.1:11
icmp 203.0.113.2:12    10.2.99.100:12     203.0.113.1:12     203.0.113.1:12
icmp 203.0.113.2:9     10.2.99.100:9      203.0.113.1:9      203.0.113.1:9
icmp 203.0.113.5:13    10.1.10.20:13      203.0.113.1:13     203.0.113.1:13
icmp 203.0.113.5:14    10.1.10.20:14      203.0.113.1:14     203.0.113.1:14
icmp 203.0.113.5:15    10.1.10.20:15      203.0.113.1:15     203.0.113.1:15
icmp 203.0.113.5:16    10.1.10.20:16      203.0.113.1:16     203.0.113.1:16
icmp 203.0.113.6:5     10.1.30.20:5       203.0.113.1:5      203.0.113.1:5
icmp 203.0.113.6:6     10.1.30.20:6       203.0.113.1:6      203.0.113.1:6
icmp 203.0.113.6:7     10.1.30.20:7       203.0.113.1:7      203.0.113.1:7
icmp 203.0.113.6:8     10.1.30.20:8       203.0.113.1:8      203.0.113.1:8
---  203.0.113.3       10.2.200.10        ---                ---
```
PC7 (Office B) wird auf die Adresse von R0 G0/2 (203.0.113.2) übersetzt. PC1 und PC5 (Office A) bekommen je eine eigene Adresse aus dem Pool (.5 und .6).

```
R0# show ip nat statistics
Total translations: 1 (1 static, 0 dynamic, 0 extended)
Outside Interfaces: GigabitEthernet0/2
Inside Interfaces: GigabitEthernet0/0 , GigabitEthernet0/1
Hits: 25  Misses: 28
Expired translations: 28
Dynamic mappings:
-- Inside Source
access-list 2 pool OfficeA refCount 0
 pool OfficeA: netmask 255.255.255.240
       start 203.0.113.4 end 203.0.113.14
       type generic, total addresses 11 , allocated 0 (0%), misses 0
```

Statisches NAT, Ping vom ISP auf die öffentliche Adresse von S-B1:
```
ISP# ping 203.0.113.3
Sending 5, 100-byte ICMP Echos to 203.0.113.3, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5)
```

_L-B1 (PAT über das WLAN): folgt mit Phase 06._

### SSH

PC4 (Office A) zu R0:
```
C:\>ssh -l ccna 10.255.0.1
Password:
R0>en
Password:
R0#
```

PC7 (Office B) zu ASW-A4 in Office A, über den WAN-Link:
```
C:\>ssh -l ccna 10.1.99.14
Password:
ASW-A4>en
Password:
ASW-A4#
```

## Probleme und Learnings

- **Fehlendes Default-Gateway auf den ASW in Office B:** Ping von ASW-B1/B2 auf 10.255.0.1 schlug fehl, weil `ip default-gateway 10.2.99.1` fehlte. Nachgetragen.
- **Sommerzeit:** `clock summer-time` gibt es in Packet Tracer nicht. Im Lab gilt nur `clock timezone CET 1`, die Sommerzeit-Umstellung ist nicht abgebildet (auf echter Hardware wäre sie nötig).
- **`clock set` in Packet Tracer:** Die eingegebene Zeit wird als UTC interpretiert, nicht als Ortszeit wie auf echter Hardware. Deshalb zeigt `show ntp status` UTC und `show clock` CET (UTC+1). Die Log-Zeitstempel zeigen UTC.
- **NTP-Quelladresse:** `ntp source` gibt es in Packet Tracer nicht. Die Antworten von R0 kommen dadurch von der Interface-Adresse (z. B. 10.0.0.5) statt von der Loopback 10.255.0.1, die Synchronisation funktioniert trotzdem. Auf echter Hardware würde man `ntp source loopback 0` setzen.
- **Calendar auf Switches:** `ntp update-calendar` wird auf den 2960-Access-Switches (ASW) nicht akzeptiert, `ntp ?` bietet dort nur `authenticate`, `authentication-key`, `master`, `server` und `trusted-key`. Die ASW synchronisieren ihre Uhr trotzdem per `ntp server`, nur das Hardware-Calendar wird nicht nachgeführt.
- **SSH-Version:** Ohne `ip ssh version 2` meldet `show ip ssh` die Version 1.99, das Gerät akzeptiert dann SSH 1 und 2. Version 1 gilt als unsicher, deshalb wird Version 2 erzwungen. Voraussetzung ist ein RSA-Schlüssel ab 768 Bit (im Lab 2048).
- **DHCP-Server S-B1:** Der DHCP-Dienst steht in Packet Tracer standardmäßig auf Off. Das Relay brachte die Anfrage bis zu S-B1, aber ohne Antwort (Simulationsmodus: Paket kommt an, geht nicht weiter). Zusätzlich braucht der Server ein Default-Gateway (10.2.200.1), sonst kann er die Antwort an den Relay-Router in einem anderen Subnetz nicht zurückschicken. Der Standardpool `serverPool` lässt sich nicht entfernen, er stört nicht.
- **Statisches NAT für S-B1:** Mit 203.0.113.3 ist der interne Infrastruktur-Server (DHCP, DNS, Syslog, TFTP) aus dem Internet erreichbar. Das ist reine Prüfungsübung. Im produktiven Netz wäre das eine unnötige Angriffsfläche, ein erreichbarer Server wäre dort ein eigener, abgeschotteter Host.
- **Internet-Wolke:** Zwischen R0 und ISP lag eine Cloud-PT. Der Ping über den Link scheiterte, beide Interfaces waren up/up, die Zähler zeigten aber `0 packets input` auf beiden Seiten. Die Wolke hatte keine Zuordnung zwischen den Ports und leitete nichts weiter. Lösung: Wolke entfernt, R0 G0/2 direkt mit ISP G0/0 verbunden (Crossover).
- **AP in VLAN 99, Clients in VLAN 50:** Der Port des LWAP (ASW-B2 G0/2) ist ein Access-Port in VLAN 99, der AP braucht nur eine Management-IP für den CAPWAP-Tunnel zum WLC. Der Verkehr der WLAN-Clients läuft im Tunnel zum WLC und wird erst dort in VLAN 50 ausgegeben (Trunk ASW-B2 G0/1, VLAN 50 und 99). Deshalb braucht der AP-Port VLAN 50 nicht, und die DHCP-Relays auf R-B1 stehen auf G0/1.50 (Clients) und G0/1.99 (AP).
