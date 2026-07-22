# IPv6 Präfix Aufteilung

**User:** Anonymous  
**Created:** 2026/7/20 16:18:46  
**Updated:** 2026/7/20 18:31:09  
**Exported:** 2026/7/20 18:33:11  
**Link:** [<https://chatgpt.com/c/6a5e2dd9-b9c4-83eb-9153-edd01747270d>](https://chatgpt.com/c/6a5e2dd9-b9c4-83eb-9153-edd01747270d)

## Prompt:

7/20/2026, 4:18:46 PM

ich brauche hardware zum verteilen von einem gemeinsamen internetanschluss. also einen managed switch? der internetanschluss ist ein glasfaseranschluss (natives IPv6), also der anschluss hat einen IPv6-präfix. jetzt brauche ich ein stück hardware (managed switch?), der aus dem einen IPv6-präfix mehrere IPv6-präfixe macht, so dass jeder nutzer von diesem internetanschluss einen eindeutigen IPv6-präfix hat. geht das?

## Response:

7/20/2026, 4:18:46 PM

Kurz gesagt: **Nein, ein Managed Switch kann das nicht.** Du brauchst einen **Router**, nicht einen Switch.

Der entscheidende Punkt ist, dass ein IPv6-Präfix **nicht einfach in mehrere Präfixe aufgeteilt werden kann**, ohne dass der Router das Routing übernimmt.

### Wie funktioniert das bei IPv6?

Angenommen, dein Provider gibt dir:

- `2001:db8:1234::/56`

Dann enthält dieser /56 **256 einzelne /64-Netze**:

- `2001:db8:1234:0000::/64`
- `2001:db8:1234:0001::/64`
- `2001:db8:1234:0002::/64`
- ...
- `2001:db8:1234:00ff::/64`

Jeder Nutzer bekommt ein eigenes /64. Das ist der Standard bei IPv6.

Der Router übernimmt dann:

- WAN: Verbindung zum Glasfaseranschluss
- LAN1 → `2001:db8:1234:1::/64`
- LAN2 → `2001:db8:1234:2::/64`
- LAN3 → `2001:db8:1234:3::/64`
- usw.

------------------------------------------------------------------------

## Welche Hardware kann das?

Fast jeder professionelle Router:

- MikroTik
- Ubiquiti EdgeRouter / UniFi Gateway
- OPNsense
- pfSense
- Cisco
- Juniper

kann mehrere IPv6-Präfixe aus einem delegierten Präfix vergeben.

------------------------------------------------------------------------

## Managed Switch vs. Router

Ein Managed Switch kann:

- VLANs
- QoS
- Port Isolation
- STP
- LACP

Er kann **kein IPv6-Routing** zwischen verschiedenen Präfixen übernehmen.

------------------------------------------------------------------------

## Wenn jeder Nutzer komplett getrennt sein soll

Dann sieht das ungefähr so aus:

                     Glasfaser
                         │
                    Provider-Router
                         │
                  Router (deiner)
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       VLAN10         VLAN20         VLAN30
          │              │              │
       Nutzer A      Nutzer B      Nutzer C

Der Router weist zu:

- VLAN10 → `...:10::/64`
- VLAN20 → `...:20::/64`
- VLAN30 → `...:30::/64`

Der Managed Switch transportiert lediglich die VLANs.

------------------------------------------------------------------------

## Wenn der Provider nur ein /64 gibt

Dann wird es schwierig.

Mit nur

    2001:db8:1234::/64

kannst du **nicht** mehrere /64 für mehrere Kunden bilden. Ein /64 ist praktisch die kleinste sinnvolle Einheit eines IPv6-LANs.

Dann gibt es nur wenige Möglichkeiten:

- NAT66 (nicht empfehlenswert)
- NDP-Proxy
- Bridge-Lösungen
- vom Provider ein größeres Präfix (/60, /56 oder /48) anfordern

Für mehrere unabhängige Nutzer sollte der Provider mindestens ein **/56** (oder besser **/48**) delegieren.

------------------------------------------------------------------------

## Für wie viele Nutzer?

Wie viele Nutzer bzw. Wohnungen sollen versorgt werden?

Und noch wichtiger:

**Welches IPv6-Präfix stellt dein Provider tatsächlich bereit?**

Zum Beispiel:

- /64
- /60
- /56
- /48

Davon hängt ab, welche Hardware sinnvoll ist und ob das Vorhaben überhaupt ohne Tricks möglich ist.

## Prompt:

7/20/2026, 4:21:31 PM

> Welches IPv6-Präfix stellt dein Provider tatsächlich bereit?

das weiss ich noch nicht. es ist ein glasfaseranschluss bei "deutsche telekom".

> Für wie viele Nutzer?

12 bis 24

## Response:

7/20/2026, 4:21:32 PM

Für **12–24 Nutzer** ist das mit IPv6 grundsätzlich überhaupt kein Problem – **wenn** die Telekom ein ausreichend großes Präfix delegiert.

### Was vergibt die Telekom?

Bei Telekom-Glasfaser (Privat- und viele Geschäftsanschlüsse) wird üblicherweise per DHCPv6 Prefix Delegation ein **/56** vergeben. Das ist in der Praxis der häufigste Fall, allerdings hängt es vom Tarif und Anschluss ab.

Ein **/56** ergibt:

- 256 × /64-Netze

Du benötigst für 24 Nutzer lediglich 24 × /64. Das wäre also mehr als ausreichend.

### Welche Hardware brauchst du?

Ich würde das in zwei Komponenten aufteilen:

    Glasfaser
        │
    ONT (oder Glasfasermodem)
        │
    IPv6-Router
        │
    Managed Switch
        │
    12–24 Anschlüsse/VLANs

Der **Router** macht:

- DHCPv6 Prefix Delegation vom Provider
- Aufteilung des /56 in einzelne /64
- Routing
- Firewall
- Router Advertisements (RA)

Der **Managed Switch** macht:

- VLANs
- Port-Zuordnung
- ggf. QoS
- Port-Isolation

### Welche Router eignen sich?

Für 24 Nutzer würde ich eher professionelle Hardware einsetzen:

- **MikroTik RB5009** – sehr leistungsfähig, gutes Preis-Leistungs-Verhältnis.
- **MikroTik CCR2004** – wenn später mehr Bandbreite oder Nutzer geplant sind.
- **pfSense** oder **OPNsense** auf einem kleinen x86-System – sehr flexibel und komfortabel.
- **Ubiquiti UniFi Gateway** – wenn du ohnehin das UniFi-Ökosystem nutzen möchtest.

Für ein Mehrnutzer-Netz würde ich persönlich **MikroTik** oder **OPNsense** bevorzugen.

### Soll jeder Nutzer sein eigenes Netz haben?

Das ist die übliche Vorgehensweise:

    Kunde 1  → VLAN 101 → 2001:db8:...:101::/64
    Kunde 2  → VLAN 102 → 2001:db8:...:102::/64
    ...
    Kunde 24 → VLAN 124 → 2001:db8:...:124::/64

Dann können die Nutzer sich gegenseitig nicht direkt erreichen (Firewall-Regeln vorausgesetzt), obwohl alle denselben Internetanschluss verwenden.

### Ein wichtiger Punkt: IPv4

Du hast erwähnt, dass der Anschluss **natives IPv6** hat. Viele Anwendungen benötigen aber weiterhin IPv4. Daher solltest du vorab klären:

- Gibt es **Dual Stack** (IPv4 + IPv6)?
- Oder nur **IPv6**?
- Oder **DS-Lite**?

Falls nur IPv6 vorhanden ist, benötigen IPv4-Dienste einen Übersetzungsmechanismus (z. B. NAT64/464XLAT oder einen vorgeschalteten IPv4-Tunnel). Das sollte bei der Planung berücksichtigt werden.

### Eine Rückfrage

Handelt es sich um ein **Mehrfamilienhaus**, ein **Büro**, ein **Studentenwohnheim** oder etwas anderes?

Davon hängt ab, ob jeder Nutzer nur Internetzugang erhalten soll oder zusätzlich komplett voneinander isoliert sein muss (inklusive Schutz vor Broadcasts, Multicast und direktem Layer-2-Verkehr). Das beeinflusst die empfohlene Netzarchitektur.

## Prompt:

7/20/2026, 4:24:52 PM

> Handelt es sich um ein Mehrfamilienhaus, ein Büro, ein Studentenwohnheim oder etwas anderes?

ein wohnblock mit 24 wohnungen.  
idealerweise sollten die wohnungen direkt miteinander kommunizieren können.  
aber für die legale absicherung (täter-identifikation bei online-kriminalität) muss jede wohnung einen eindeutigen IPv6-präfix haben, also eine öffentliche IP-adresse muss man eindeutig zuordnen können zu einer wohnung

## Response:

7/20/2026, 4:24:52 PM

Das ist ein sehr interessanter Anwendungsfall. Rein technisch ist das gut lösbar – **rechtlich und organisatorisch** gibt es aber ein paar Punkte zu beachten.

## Technisch

Ja, genau dafür ist IPv6 eigentlich gemacht.

Angenommen, die Telekom delegiert dir ein **/56**:

    2001:db8:1234:0000::/56

Dann vergibst du beispielsweise:

    Wohnung 01 → 2001:db8:1234:0001::/64
    Wohnung 02 → 2001:db8:1234:0002::/64
    ...
    Wohnung 24 → 2001:db8:1234:0018::/64

Jede Wohnung hat dann ihren **eigenen öffentlichen Adressbereich**. Anhand der Quell-IP lässt sich eindeutig erkennen, aus welcher Wohnung der Verkehr stammt.

Das ist einer der großen Vorteile von IPv6 gegenüber IPv4 mit NAT.

------------------------------------------------------------------------

## Routing

Du brauchst keinen Router pro Wohnung.

Ein einzelner Router kann problemlos 24 Netze verwalten:

    Internet
        │
    Telekom
        │
    Router
        │
    Managed Switch
     ├── Port 1 → Wohnung 1 (/64)
     ├── Port 2 → Wohnung 2 (/64)
     ├── ...
     └── Port24 → Wohnung24 (/64)

Oder – falls mehrere Switches im Gebäude verteilt sind – über VLANs.

------------------------------------------------------------------------

## Kommunikation zwischen Wohnungen

Das ist der interessante Teil.

Du schreibst:

> idealerweise sollten die wohnungen direkt miteinander kommunizieren können.

Das geht.

Normalerweise sperrt die Firewall eines Routers den Verkehr zwischen den /64-Netzen.

Du könntest stattdessen erlauben:

    Wohnung 1 <-----> Wohnung 2
    Wohnung 1 <-----> Wohnung 3
    ...

also Inter-VLAN-Routing zulassen.

Die Wohnungen hätten trotzdem unterschiedliche Präfixe.

------------------------------------------------------------------------

## Täteridentifikation

Hier muss man unterscheiden:

### IPv6

Bei IPv6 funktioniert die Zuordnung sehr gut.

Wenn in den Logs steht

    2001:db8:1234:0007:....

weißt du sofort:

> Wohnung 7

Das setzt allerdings voraus, dass die Präfixe **statisch** den Wohnungen zugewiesen sind oder dass du Änderungen sauber protokollierst.

------------------------------------------------------------------------

### IPv4

Falls alle Wohnungen **eine gemeinsame öffentliche IPv4-Adresse** verwenden (Carrier NAT oder normales NAT), reicht die IPv4-Adresse allein **nicht** zur Identifikation. Dann müsstest du zusätzlich NAT-Logs führen (Zeitstempel, Ports, interne Adresse), was aufwendig werden kann.

Deshalb wäre ein echter Dual-Stack-Anschluss mit eigener öffentlicher IPv4 je Wohnung ideal – ist aber bei einem gemeinsamen Anschluss praktisch nicht verfügbar.

------------------------------------------------------------------------

## Meine Empfehlung

Für einen Wohnblock mit 24 Wohnungen würde ich so vorgehen:

- Telekom-Glasfaser mit möglichst einem **/56 oder /48**
- leistungsfähiger Router (z. B. MikroTik oder OPNsense)
- Managed Switch(es)
- jede Wohnung bekommt
  - ein eigenes VLAN
  - ein eigenes IPv6-/64
- Firewall-Regeln nach Bedarf (Kommunikation zwischen Wohnungen erlauben oder einschränken)

Das ist eine saubere und gut skalierbare Lösung.

## Ein rechtlicher Hinweis

Wenn du den Anschluss für Dritte bereitstellst (24 Wohnungen), bewegst du dich faktisch in der Rolle eines Internetzugangsanbieters für deine Bewohner. Die technische Zuordnung über feste IPv6-Präfixe ist dafür hilfreich, ersetzt aber nicht automatisch alle rechtlichen oder datenschutzrechtlichen Anforderungen. Je nach Ausgestaltung (z. B. Vermieter, Genossenschaft oder kommerzieller Anbieter) können weitere Pflichten relevant sein.

**Mich würde noch etwas interessieren:** Ist geplant, dass jede Wohnung **genau einen Ethernet-Port** bekommt, an den der Bewohner seinen eigenen Router anschließt? Oder möchtest du zusätzlich WLAN oder mehrere LAN-Dosen pro Wohnung bereitstellen? Das beeinflusst die Netzarchitektur und die Hardwareauswahl.

## Prompt:

7/20/2026, 4:28:23 PM

> Ist geplant, dass jede Wohnung genau einen Ethernet-Port bekommt

ja, wir müssen ja extra netzwerkkabel legen, und da haben wir genug arbeit wenn jede wohnung "nur" ein kabel hat... also die zuordnung "LAN-port -\> netzwerkkabel -\> wohnung" ist eindeutig

> bewegst du dich faktisch in der Rolle eines Internetzugangsanbieters für deine Bewohner

ja, der anschlussinhaber wäre dann ein ISP. was heisst das? vorratsdatenspeicherung?

> Genossenschaft

der hauseigentümer ist eine genossenschaft, und idealerweise ist die genossenschaft dann auch der anschlussinhaber, so dass der internetanschluss nicht abhängig ist von einem mieter

## Response:

7/20/2026, 4:28:23 PM

Das klingt nach einem sehr sinnvollen Konzept. Tatsächlich entspricht das dem Aufbau vieler kleiner Campus-, Wohnheim- oder Genossenschaftsnetze.

## Technisch würde ich es so machen

Jede Wohnung bekommt genau einen Port:

``` text
                  Telekom-Glasfaser
                         │
                    ONT / Glasfasermodem
                         │
                Router (IPv6-Routing)
                         │
                  Managed Switch
        ┌─────────┬─────────┬─────────┐
        │         │         │
     Port 1    Port 2    Port 24
        │         │         │
   Wohnung 1 Wohnung 2 Wohnung 24
```

Jeder Port gehört zu einem eigenen VLAN.

Der Router routet beispielsweise:

- VLAN 101 → Wohnung 1 → `/64`
- VLAN 102 → Wohnung 2 → `/64`
- …
- VLAN 124 → Wohnung 24 → `/64`

Der Bewohner kann dann an seiner LAN-Dose einen beliebigen Router (oder direkt einen PC) anschließen.

------------------------------------------------------------------------

## Das Schöne an IPv6

Du brauchst **kein NAT**.

Jede Wohnung erhält ein eigenes öffentliches Netz.

Wenn irgendwann Wohnung 17 Mist baut, sieht man beispielsweise:

    2001:db8:1234:11::abcd

→ gehört zum Präfix der Wohnung 17.

Die Zuordnung ist dauerhaft eindeutig, solange die Präfixe fest vergeben sind.

------------------------------------------------------------------------

## Rechtlich

Hier wird es etwas komplex.

Allein dadurch, dass die Genossenschaft den Anschluss betreibt, wird sie **nicht automatisch** wie ein großer ISP mit allen regulatorischen Pflichten eines Telekommunikationsunternehmens behandelt. Welche Pflichten bestehen, hängt unter anderem davon ab,

- ob der Internetzugang nur den eigenen Mitgliedern/Mietern bereitgestellt wird,
- ob dafür ein separates Entgelt verlangt wird,
- und wie das Angebot organisatorisch ausgestaltet ist.

Die rechtliche Einordnung sollte deshalb vor der Umsetzung geprüft werden.

### Vorratsdatenspeicherung

Die klassische deutsche Vorratsdatenspeicherung ist seit Jahren Gegenstand gerichtlicher Entscheidungen und in der ursprünglich vorgesehenen Form derzeit nicht allgemein anwendbar. Man sollte also nicht davon ausgehen, dass ein kleiner Wohnungsanbieter automatisch umfangreiche Verkehrsdaten speichern muss.

Unabhängig davon gilt: Wenn Behörden einen konkreten Anschluss ermitteln möchten, muss eine Zuordnung grundsätzlich möglich sein. Mit **statisch zugewiesenen IPv6-Präfixen** ist diese Zuordnung technisch sehr einfach.

------------------------------------------------------------------------

## Ich würde die Präfixe statisch vergeben

Nicht dynamisch.

Also beispielsweise dauerhaft:

| Wohnung | VLAN | IPv6           |
|---------|------|----------------|
| 1       | 101  | `...:101::/64` |
| 2       | 102  | `...:102::/64` |
| …       | …    | …              |
| 24      | 124  | `...:124::/64` |

Dann musst du praktisch nichts protokollieren, weil sich die Zuordnung nie ändert.

------------------------------------------------------------------------

## Hardware

Für 24 Wohnungen würde ich eher keine Consumer-Hardware einsetzen.

Eine solide Kombination wäre:

- **Router:** MikroTik RB5009 oder ein kleiner x86-Rechner mit OPNsense/pfSense.
- **Switch:** 24-Port-Managed-Switch mit VLAN-Unterstützung (z. B. MikroTik, TP-Link Omada, Ubiquiti oder Netgear).

Das ist preislich überschaubar und professionell genug für einen Dauerbetrieb.

## Ein weiterer Gedanke

Da ihr ohnehin **jedes Netzwerkkabel neu verlegt**, würde ich überlegen, **zwei Kabel pro Wohnung** zu ziehen, auch wenn zunächst nur eines genutzt wird.

Der Materialpreis für ein zweites Cat-6A- oder Cat-7-Kabel ist meist gering im Vergleich zu den Arbeitskosten. Ein Reservekabel ist später Gold wert – etwa bei einem Defekt oder wenn irgendwann ein zweiter Anschluss (z. B. separates Verwaltungsnetz, IP-Telefonie oder ein zweiter Internetanschluss) benötigt wird. Gerade wenn Wände später nicht mehr geöffnet werden sollen, lohnt sich diese Reserve oft.

## Prompt:

7/20/2026, 4:33:39 PM

> Switch: 24-Port-Managed-Switch mit VLAN-Unterstützung

aah, also doch mit einem Managed-Switch, zur übersetzung zwischen VLAN-ID und ethernet-port, aber die VLANs werden verwaltet vom router. aber trotzdem kein NAT, also effizienter als ein alter VDSL-anschluss mit NAT

> ob dafür ein separates Entgelt verlangt wird

nein, die kosten für den internetanschluss sind teil der nebenkosten, weil jede wohnung wir "mit internetanschluss" vermietet. aber das macht doch keinen unterschied, oder?

## Response:

7/20/2026, 4:33:39 PM

Genau – du hast den technischen Aufbau richtig verstanden.

### Technisch

Der Managed Switch macht im Wesentlichen nur Layer 2:

- Port 1 → VLAN 101
- Port 2 → VLAN 102
- …
- Port 24 → VLAN 124

Er routet nichts (zumindest nicht in diesem Design). Das übernimmt der Router.

Der Router hat dann beispielsweise 24 VLAN-Interfaces:

``` text
eth0.101  → 2001:db8:...:101::/64
eth0.102  → 2001:db8:...:102::/64
...
eth0.124  → 2001:db8:...:124::/64
```

Jedes Interface sendet Router Advertisements (RA) und ggf. DHCPv6. Die Endgeräte in der Wohnung konfigurieren sich dann automatisch.

**NAT brauchst du für IPv6 überhaupt nicht.**

------------------------------------------------------------------------

### Warum ist das effizienter?

Bei IPv4 mit NAT passiert für jedes Paket:

    192.168.1.42:54321
            ↓
    NAT
            ↓
    203.0.113.5:40001

Der Router muss Zustände (State) verwalten und Ports umschreiben.

Bei IPv6 passiert dagegen einfach:

    2001:db8:1234:101::42
            ↓
    Routing
            ↓
    Internet

Der Router schaut nur in seine Routingtabelle und leitet das Paket weiter. Die Firewall ist zwar weiterhin zustandsbehaftet (Stateful Firewall), aber die Adressen selbst werden nicht übersetzt. Das macht das Netz konzeptionell einfacher.

------------------------------------------------------------------------

### Zur rechtlichen Frage

> Die Kosten für den Internetanschluss sind Teil der Nebenkosten.

Das **kann** durchaus einen Unterschied machen, aber nicht im Sinne von "dann gilt gar nichts". Die rechtliche Einordnung hängt nicht nur davon ab, ob ein separater Vertrag oder ein separates Entgelt existiert. Es spielen mehrere Faktoren eine Rolle, etwa:

- Wird der Dienst ausschließlich den eigenen Mietern bereitgestellt?
- Ist er Bestandteil des Mietverhältnisses?
- Erfolgt die Bereitstellung gewerblich für die Allgemeinheit oder nur innerhalb der Wohnanlage?

Ein genossenschaftlich betriebenes Hausnetz kann deshalb anders eingeordnet werden als ein kommerzieller ISP. Für eine belastbare Einschätzung solltet ihr das vorab mit einem fachkundigen Berater oder Anwalt klären.

------------------------------------------------------------------------

## Eine Idee, die gut zu eurem Projekt passen könnte

Da die Genossenschaft Eigentümer des Gebäudes ist, würde ich die Infrastruktur möglichst **dauerhaft und dokumentiert** aufbauen:

- Jede Wohnung bekommt eine feste VLAN-ID.
- Jede Wohnung bekommt dauerhaft dasselbe IPv6-/64.
- Die Port-Zuordnung wird dokumentiert.
- Die Patchfelder werden beschriftet.
- Die Zuordnung Wohnung ↔ Port ↔ VLAN ↔ IPv6-Präfix bleibt unverändert.

Das hat mehrere Vorteile:

- Fehlersuche wird einfacher.
- Ein Routertausch ändert nichts an der Adressierung.
- Die Zuordnung einer IPv6-Adresse zu einer Wohnung ist jederzeit nachvollziehbar.
- Ein Mieterwechsel erfordert keine Änderungen an der Netzkonfiguration.

Das ist genau die Art von einfacher, robuster Netzarchitektur, die man typischerweise für Gebäude mit vielen Wohneinheiten anstrebt.

## Prompt:

7/20/2026, 4:36:07 PM

> Wird der Dienst ausschließlich den eigenen Mietern bereitgestellt?

ja

> Ist er Bestandteil des Mietverhältnisses?

ja, der internetanschluss gehört fest zur wohnung, also ist nicht optional

> Erfolgt die Bereitstellung gewerblich für die Allgemeinheit oder nur innerhalb der Wohnanlage?

nur innerhalb der Wohnanlage, nur für die mieter der wohnungen, also kein öffentlicher zugang, und kein öffentliches WLAN oder so

## Response:

7/20/2026, 4:36:08 PM

Unter diesen Randbedingungen sieht das deutlich anders aus als ein klassischer ISP.

Was ihr beschreibt, ist eher ein **internes Hausnetz mit Internet-Uplink**:

- Eigentümerin/Betreiberin: die Genossenschaft
- Nutzer: ausschließlich die eigenen Mieter
- Bestandteil: Teil der Wohnungsausstattung bzw. des Mietverhältnisses
- keine Vermarktung nach außen
- kein öffentlich zugänglicher Dienst

Das ist technisch und organisatorisch näher an einem Firmen- oder Campusnetz als an einem Internetprovider, der Kunden am Markt gewinnt.

## Trotzdem: zwei Ebenen trennen

### 1. Vertrag mit der Telekom

Der Anschluss läuft auf die Genossenschaft. Das ist aus meiner Sicht der saubere Weg:

- Der Glasfaseranschluss gehört dauerhaft zur Immobilie.
- Ein Mieterwechsel ändert nichts.
- Die Infrastruktur bleibt Eigentum der Genossenschaft.

Wichtig wäre nur zu prüfen, ob der gewählte Telekom-Tarif eine solche Nutzung zulässt. Manche Privatkundentarife haben Einschränkungen bezüglich Weitergabe an Dritte bzw. "Überlassung". Ein Geschäftskundentarif oder ein spezieller Immobilien-/Mehrnutzeranschluss kann je nach Situation passender sein.

------------------------------------------------------------------------

### 2. Zuordnung bei Missbrauch

Euer IPv6-Design ist dafür hervorragend geeignet.

Beispiel:

    Wohnung 17
    VLAN 117
    IPv6-Präfix 2001:db8:1234:117::/64

Wenn eine Anfrage kommt:

> "Wer hatte die Adresse 2001:db8:1234:117::abcd?"

kann die Genossenschaft anhand der festen Dokumentation sagen:

> "Das Präfix gehört Wohnung 17."

Das ist deutlich einfacher als ein gemeinsamer NAT-Gateway mit IPv4.

------------------------------------------------------------------------

## Eine kleine technische Feinheit: Präfix statt einzelne IP-Adresse

Ein häufiger Denkfehler ist "jede Wohnung braucht eine eigene öffentliche IP".

Bei IPv6 ist es besser:

Nicht:

    Wohnung 1 → 2001:db8::1234

sondern:

    Wohnung 1 → 2001:db8:1234:0011::/64

Die Wohnung bekommt also ein ganzes Netz.

Warum?

- Smartphones, Laptops, Fernseher, IoT-Geräte usw. brauchen eigene Adressen.
- IPv6 ist genau dafür ausgelegt.
- Die Zuordnung bleibt trotzdem eindeutig.

------------------------------------------------------------------------

## Eine Sache würde ich noch ergänzen: Firewall-Regeln

Du hattest gesagt:

> idealerweise sollten die Wohnungen direkt miteinander kommunizieren können.

Das geht, aber ich würde es bewusst entscheiden.

Eine mögliche Konfiguration:

### Variante A: wie ein gemeinsames Hausnetz

    Wohnung 1 ↔ Wohnung 2 ↔ Wohnung 3

Vorteil:

- einfach
- Geräte können direkt kommunizieren

Nachteil:

- ein kompromittiertes Gerät in Wohnung A kann Geräte in Wohnung B angreifen

------------------------------------------------------------------------

### Variante B: Standardmäßig getrennt, gezielt erlauben

    Wohnung 1 ─┐
    Wohnung 2 ─┼── Internet
    Wohnung 3 ─┘

Firewall blockiert:

    Wohnung 1 → Wohnung 2

Erlaubt aber:

    Wohnung 1 → Internet
    Wohnung 1 → eigene Geräte

Das ist die häufigere professionelle Variante.

------------------------------------------------------------------------

## Noch ein Punkt zur Verkabelung

Bei 24 Wohnungen würde ich tatsächlich überlegen:

- **Cat 6A**
- kleines 19"-Rack im Technikraum
- Patchpanel
- Switch(es)
- USV für Router + Switch

Und wie erwähnt: Wenn ihr schon Kabel zieht, **zwei Kabel pro Wohnung** wären sehr wertvoll. Nicht wegen des aktuellen Internets, sondern weil ihr sonst später wieder Wände öffnen müsst.

------------------------------------------------------------------------

Eine technische Frage ist noch entscheidend:  
Wird der Glasfaseranschluss **direkt in den Technikraum der Genossenschaft** gelegt (ONT dort), oder bekommt jede Wohnung ihren eigenen Glasfaserabschluss und ihr wollt erst dahinter bündeln?

Für euren Ansatz wäre ein zentraler Übergabepunkt im Technikraum die deutlich bessere Variante.

## Prompt:

7/20/2026, 4:38:43 PM

> Wichtig wäre nur zu prüfen, ob der gewählte Telekom-Tarif eine solche Nutzung zulässt. Manche Privatkundentarife haben Einschränkungen bezüglich Weitergabe an Dritte bzw. "Überlassung". Ein Geschäftskundentarif oder ein spezieller Immobilien-/Mehrnutzeranschluss kann je nach Situation passender sein.

ja, das kann gut sein... kannst du den vertrag prüfen? der vertrag ist: Telekom Glasfaser 1000 (Upload 500):  69,95€/Monat

------------------------------------------------------------------------

<https://www.telekom.de/ueber-das-unternehmen/verbraucherinformationen>

Verbraucherinformationen

Gesetzliche Informationen zu Ihren Rechten als Verbraucher nach den Vorschriften über Verträge im Fernabsatz und im elektronischen Geschäftsverkehr.

Vertragspartner  
Bestellablauf  
Lieferung  
Versandkosten  
Zahlung  
Gewährleistung  
Widerrufsrecht  
Informationen zu Festnetz- und Mobilfunktarifen  
Speicherung der Vertragsdaten  
Vertragssprache  
Beanstandungen  
Schlichtungsstelle  
Hinweise für Ihre Elektrogeräte/Batterie Rückgabe

1.  Vertragspartner  
    Vertragspartner der im Shop angebotenen Produkte ist, soweit nicht anders angegeben, die Telekom Deutschland GmbH, Landgrabenweg 149, 53227 Bonn.

Für Anliegen im Bereich Festnetz erreichen Sie uns unter der E- Mail- Adresse <auftrag.festnetz@telekom.de> und telefonisch unter

Kundenservice Festnetz: 0800 33 01000  
aus dem Ausland: +49 180 233 1000\*

Für Anliegen im Bereich Mobilfunk erreichen Sie uns unter der E- Mail-Adresse <auftrag.mobilfunk@telekom.de> und telefonisch unter

Kundenservice Mobilfunk: 0800 33 02202,  
vom Handy: 2202  
kostenlos aus dem Ausland: +49 171 252 2088

Alternativ können Sie natürlich auch über unser Kontaktformular oder per Brief unter der oben genannten Postanschrift mit uns in Verbindung treten.

Weitere Details über uns finden Sie unter: [www.telekom.de/impressum](http://www.telekom.de/impressum).

2.  Bestellablauf  
    Sie können in diesem Online- Shop Produkte auswählen und diese über den Bestellbutton unverbindlich in den Warenkorb legen. Den Inhalt des Warenkorbs können Sie jederzeit durch Klicken des Buttons „Warenkorb“ ansehen. Sie können die Produkte auch jederzeit durch Klicken des Buttons „Löschen“ wieder aus dem Warenkorb entfernen. Nachdem Sie alle Produkte ausgewählt haben, werden diese zusammengefasst dargestellt. Mit einem Klick auf „Zur Kasse gehen“ werden weitere für den Vertragsschluss erforderliche Daten und Einwilligungen abgefragt. Danach wird eine Übersicht der ausgewählten Leistungen sowie der eingegebenen Daten angezeigt. Diese kann nochmals korrigiert werden. Erst durch Anklicken des Buttons „Zahlungspflichtig bestellen“ geben Sie ein verbindliches Bestellangebot ab. Der Vorgang lässt sich jederzeit durch Schließen des Browser-Fensters abbrechen.

Unmittelbar nach dem Absenden der Bestellung erhalten Sie eine automatisch erstellte E-Mail, die den Eingang Ihrer Bestellung im Online-Shop bestätigt. Diese stellt jedoch noch keine Vertragsannahme durch die Telekom dar.

Das Vertragsverhältnis kommt erst mit Zugang der „Auftragsbestätigung“ zustande. Diese erhalten Sie in der Regel per E-Mail, alternativ auch als Brief. Spätestens kommt das Vertragsverhältnis jedoch bei Dienstleistungen mit Bereitstellung der Leistung (z. B. Schaltung des Festnetzanschlusses oder Aktivierung der SIM-Karte durch die Telekom bei einem Mobilfunkanschluss), beim Kaufvertrag mit Erhalt der Ware, zustande.

Eine Bestellung im Online-Shop ist erst ab 18 Jahren möglich.

3.  Lieferung  
    Die Lieferung von Waren erfolgt nur innerhalb Deutschlands. Die Zustellung erfolgt mit DHL Paket, DHL Express, GLS oder DPD an die vom Kunden angegebene Lieferanschrift. Sofern in der Produktbeschreibung nicht anders angegeben, erfolgt eine Lieferung für Festnetz-Produkte und Mobilfunk-Produkte innerhalb von 1 - 4 Tagen. Werden Festnetz-Endgeräte gemeinsam mit einem Festnetz-Tarif bestellt (z. B. Media Receiver in Verbindung mit einem MagentaZuhause Tarif mit MagentaTV), erfolgt die Lieferung der bestellten Endgeräte erst 4 Tage vor der Bereitstellung des Tarifes.

a\) Mobilfunk-Produkte

Bitte beachten Sie bei der Bestellung von Mobilfunk-Produkten, dass die eingegebenen Daten mit denen aus Personalausweis, Reisepass oder Meldebescheinigung übereinstimmen müssen. Das ist wichtig, da Sie die Lieferung nur persönlich gegen Vorlage Ihres gültigen Ausweises entgegennehmen können. Diese Identitätsprüfung findet bei einer Vertragsverlängerung nicht statt. Während der Bestellung können Sie auswählen, wann die Lieferung erfolgen soll. Den Lieferstatus für Mobilfunk-Produkte erfahren Sie hier.

b\) Festnetz-Produkte

Bei der Beauftragung eines Festnetz-Tarifes (z. B. MagentaZuhause mit MagentaTV) können Sie uns einen Wunschtermin angeben. Nach Eingang Ihres Auftrages prüfen wir, wann wir Ihnen die Leistung bereitstellen können. Die Bereitstellungsdauer eines Festnetzanschlusses hängt von verschiedenen Faktoren ab und kann sich z.B. durch evtl. notwendige Installationsarbeiten verzögern. Vorbehaltlich notwendiger Baumaßnahmen erfolgt die Bereitstellung in der Regel innerhalb von 6 bis 14 Werktagen nach Beauftragung. Den genauen Termin entnehmen Sie bitte Ihrer Auftragsbestätigung.

b\) digitale-Produkte

Die Bereitstellung der digitalen Produkte erfolgt unmittelbar nach Abschluss der Bestellung durch Klicken des Buttons „zahlungspflichtig bestellen“ und dem Abschluss des Bezahlvorganges.

4.  Preise, Versandkosten  
    Alle Preise in unserem Online-Shop beinhalten die gesetzliche Umsatzsteuer und alle sonstigen Preisbestandteile.

Die Höhe der Versandkosten teilen wir Ihnen im Rahmen des Bestellprozesses mit.

5.  Zahlung  
    a) Mobilfunk-Produkte (Endgeräte, Zubehör):

Im Online-Shop bestellte Ware können Sie wahlweise per SEPA-Lastschriftmandat (Bankeinzug), Kreditkarte, PayPal oder per Nachnahme bezahlen.

Bei Zahlung per SEPA-Lastschriftmandat (Bankeinzug) erfolgt die Belastung Ihres Kontos ca. zwei bis drei Tage nach Auslieferung der Ware.

Bei Zahlung mit Kreditkarte wird Ihre Kreditkarte mit Erstellung der Kreditkartenrechnung belastet. Bitte beachten Sie die Bedingungen Ihres Zahlungssystemanbieters. Es werden die Anbieter Mastercard, Visa und American Express akzeptiert.

Bei Zahlung per PayPal wird während der Bestellung der Rechnungsbetrag auf Ihrem PayPal-Konto autorisiert (vorgemerkt). Sobald Ihr Gerät versandbereit ist, buchen wir den Betrag von Ihrem PayPal-Konto ab.

Bei Zahlung per Nachnahme wird der Rechnungsbetrag bei Lieferung fällig. Zusätzlich fällt eine Nachnahmegebühr an, deren Höhe wir Ihnen im Bestellprozess mitteilen. Bitte halten Sie den Rechnungsbetrag möglichst passend bereit.

Bitte beachten Sie, dass die Bezahlarten Kreditkarte und PayPal nur unter bestimmten Voraussetzungen angeboten werden. So zum Beispiel, wenn alle Artikel im Warenkorb kurzfristig lieferbar sind.

b\) Festnetz-Produkte (Endgeräte, Zubehör):

Im Online-Shop bestellte Ware können Sie wahlweise und je nachdem, ob Sie als Gast oder Kunde bestellen, per SEPA-Lastschriftmandat (Bankeinzug), über Ihre Telefonrechnung oder per Nachnahme bezahlen.

Bei Zahlung per Bankeinzug erfolgt die Belastung Ihres Kontos nicht vor dem siebten Tag nach Auslieferung der Ware.

Bei Bezahlung per Nachnahme ist der Rechnungsbetrag bei Lieferung fällig. Zusätzlich fällt eine Nachnahmegebühr an, deren Höhe wir Ihnen im Bestellprozess mitteilen. Bitte halten Sie den Rechnungsbetrag möglichst passend bereit.

c\) Festnetz- und Mobilfunktarife, sonstige Dienstleistungen, digitale Produkte

Bezüglich der beauftragten Tarife, Zubuchoptionen, sonstigen Dienstleistungen und digitalen Produkte gelten die in unseren Allgemeinen Geschäftsbedingungen (AGB) vorgesehenen Zahlungsbedingungen, denen Sie im Bestellprozess zugestimmt haben.

6.  Gewährleistung  
    Für die in unserem Shop angebotenen Waren und digitalen Produkte bestehen die gesetzlichen Mängelansprüche. Sofern der Hersteller für die Waren eine Herstellergarantie übernimmt, bestehen die gesetzlichen Mängelansprüche und die Herstellergarantie nebeneinander. Wenn Sie Fragen haben oder unseren Kundenservice kontaktieren möchten, finden Sie hier alle Kontaktwege: [www.telekom.de/kontakt](http://www.telekom.de/kontakt).

7.  Widerrufsrecht  
    Als Verbraucher steht Ihnen bei Bestellungen in unserem Shop ein 14-tägiges Widerrufsrecht zu. Nähere Details zu Ihrem Widerrufsrecht erhalten Sie im Rahmen des Bestellprozesses und unter: [www.telekom.de/widerrufsrecht](http://www.telekom.de/widerrufsrecht).

8.  Informationen zu Festnetz- und Mobilfunktarifen  
    Informationen über Festnetz- und Mobilfunktarife, die auf Grundlage von Allgemeinen Geschäftsbedingungen angeboten werden wie Informationen über Leistungen, Preise, Vertragslaufzeiten, Kündigungsbedingungen und sonstige Informationen (wie z.B. Voraussetzungen für einen Anbieterwechsel) und über Ihre Rechte als Endnutzer finden Sie in den für das jeweilige Produkt geltenden Allgemeinen Geschäftsbedingungen, Leistungsbeschreibungen und Preislisten. Diese Dokumente sind unter [www.telekom.de/agb](http://www.telekom.de/agb) und im Rahmen des Bestellprozesses abrufbar.

Die Datenübertragungsrate der bereitgestellten Festnetz-Internetverträge können Sie über die Breitbandmessung Desktop-App der Bundesnetzagentur unter [www.breitbandmessung.de](http://www.breitbandmessung.de) überprüfen.

Informationen über die tatsächliche standortbezogene Mobilfunknetzabdeckung der Telekom, einschließlich einer Kartendarstellung zur aktuellen Netzabdeckung, finden Sie im Internet unter [www.telekom.de/netz/mobilfunk-netzausbau](http://www.telekom.de/netz/mobilfunk-netzausbau)

9.  Speicherung der Vertragsdaten  
    Wir speichern Ihre Vertragsdaten und senden Ihnen die Vertragsunterlagen E-Mail und gegebenenfalls per Post zu.

Im Kundencenter ([www.telekom.de/mein-magenta-kundencenter](http://www.telekom.de/mein-magenta-kundencenter)) können Sie nach Eingabe Ihrer persönlichen Zugangsdaten den Auftragsstatus Ihres Tarifes bzw. den Bestellstatus Ihrer bestellten Ware einsehen. Ferner können Sie dort jederzeit Ihre aktuellen Vertragsdaten einsehen und ändern, sowie Ihre persönlichen Daten verwalten.

10. Vertragssprache  
    Die ausschließliche Vertragssprache ist Deutsch.

11. Beanstandungen  
    Sie haben eine Beanstandung zur Qualität unserer Dienstleistungen, zur Vertragsdurchführung oder zu Ihrer Abrechnung?

Selbstverständlich können Sie sich jederzeit an folgende Rufnummern wenden:

Bei Beanstandungen im Bereich Festnetz erreichen Sie uns unter der E- Mail- Adresse <auftrag.festnetz@telekom.de> und telefonisch unter

Kundenservice Festnetz: 0800 33 01000  
aus dem Ausland: +49 180 233 1000\*

Bei Beanstandungen im Bereich Mobilfunk erreichen Sie uns unter der E- Mail- Adresse <auftrag.mobilfunk@telekom.de> und telefonisch unter

Kundenservice Mobilfunk: 0800 33 02202,  
vom Handy: 2202  
kostenlos aus dem Ausland: +49 171 252 2088

Alternativ können Sie natürlich auch über unser Kontaktformular mit uns in Verbindung treten oder per Brief an

Telekom Deutschland GmbH  
Landgrabenweg 149  
53227 Bonn

Die Bearbeitungsdauer bei Beanstandungen zur Qualität unserer Dienstleistungen, zur Vertragsdurchführung oder zu Ihrer Abrechnung beträgt durchschnittlich 3 Arbeitstage (knapp 70% der Beschwerden bearbeiten wir innerhalb eines Tages, einen höheren Zeitaufwand benötigen wir vor allem für Beanstandungen, die aufwändige technische oder bauliche Maßnahmen (inkl. Baulicher Genehmigungen) erfordern.

12. Schlichtungsstelle  
    Nach § 68 Telekommunikationsgesetz besteht die Möglichkeit, ein Schlichtungsverfahren vor der Bundesnetzagentur einzuleiten, wenn Streit darüber besteht, ob der Anbieter von öffentlich zugänglichen Telekommunikationsdiensten bestimmte, in § 68 TKG aufgeführte Endkundenrechte nicht berücksichtigt hat. Hierzu muss ein Antrag bei der Bundesnetzagentur gestellt werden. Der Antrag ist zu richten an:

Bundesnetzagentur Ref. 216, Schlichtungsstelle

Postfach 8001

53105 Bonn

Telefax 030 224 80518

Nähere Details zum Verfahrensablauf erhalten Sie auf der Internetseite der Bundesnetzagentur [www.bundesnetzagentur.de](http://www.bundesnetzagentur.de).

13. Hinweise für Ihre Elektrogeräte/Batterie-Rückgabe  
    Hinweise nach § 18 Abs. 3 ElektroG und § 18 Abs. 1 Batteriegesetz

## Response:

7/20/2026, 4:38:46 PM

Ich habe die öffentlich auffindbaren Telekom-Unterlagen zum Tarif **Glasfaser 1.000 (1.000/500 Mbit/s)** geprüft. Die von dir verlinkte Seite selbst enthält vor allem Verbraucherinformationen; die eigentlichen Tarifdetails stehen in den Leistungsbeschreibungen/AGB-Dokumenten. Der Tarif wird als Internet- und Telefon-Flatrate für einen Glasfaseranschluss angeboten. ([Telekom](https://www.telekom.de/netz/glasfaseranschluss?utm_source=chatgpt.com))

Die entscheidende Frage ist nicht die Bandbreite, sondern die **Nutzungsart**:

> Darf ein Privatkunden-Festnetzanschluss der Telekom dazu verwendet werden, 24 Wohnungen mit Internet zu versorgen?

Dazu finde ich in den öffentlich sichtbaren Unterlagen **keine eindeutige Klausel**, die deinen konkreten Fall ausdrücklich erlaubt oder verbietet. Die Verbraucher-Leistungsbeschreibung spricht allgemein vom "Kunden" und seinem Anschluss, nicht von einer Mehrparteienversorgung innerhalb eines Gebäudes. ([Telekom](https://www.telekom.de/agb/downloads/54933.pdf?utm_source=chatgpt.com))

Meine Einschätzung:

## Technisch: ja

Der Anschluss kann das problemlos.

Für die Telekom sieht der Verkehr vermutlich einfach so aus:

    Telekom
      |
      | 1x Glasfaseranschluss
      |
    Genossenschaftsrouter
      |
      +-- Wohnung 1
      +-- Wohnung 2
      ...
      +-- Wohnung 24

Die Telekom sieht nicht "24 Kunden", sondern **einen Kunden mit vielen Endgeräten hinter einem Router**.

------------------------------------------------------------------------

## Vertraglich: hier wäre ich vorsichtig

Der Knackpunkt ist die Formulierung "Anschluss für den Kunden".

Ein normaler MagentaZuhause/Glasfaser-Tarif ist primär für einen Haushalt gedacht. Bei euch wäre der Anschlussinhaber zwar nur **eine juristische Person (Genossenschaft)**, aber die Leistung wird an 24 Haushalte weitergereicht.

Das ist nicht automatisch verboten - ein Unternehmen betreibt ja auch mit einem einzigen Internetanschluss ein ganzes Bürogebäude. Aber ein Mehrfamilienhaus mit 24 eigenständigen Mietparteien ist näher an einem "Internet-Zugangsanbieter innerhalb einer Immobilie".

Ich würde deshalb nicht einfach bestellen und hoffen, sondern die Telekom schriftlich fragen.

Eine gute Formulierung wäre:

> "Unsere Wohnungsgenossenschaft möchte einen Glasfaseranschluss als Bestandteil der Miete in einem eigenen Wohngebäude bereitstellen. Der Anschluss wird ausschließlich von den eigenen Mietern innerhalb dieser Immobilie genutzt. Es erfolgt keine öffentliche Bereitstellung und kein Weiterverkauf. Ist der Tarif Telekom Glasfaser 1000 hierfür zulässig oder benötigen wir einen Geschäftskunden-/Sondertarif?"

Wichtig: **Nicht** "ISP" oder "Provider" schreiben - das löst eventuell unnötige interne Prozesse aus. Technisch stimmt es zwar, aber im normalen Sprachgebrauch klingt das nach kommerziellem Wiederverkauf.

------------------------------------------------------------------------

## Preisvergleich

69,95 €/Monat sind für 1 Gbit/s FTTH ein normaler Endkundentarif. ([Telekom](https://www.telekom.de/netz/glasfaseranschluss?utm_source=chatgpt.com))

Für 24 Wohnungen wären das:

    69,95 € / 24 = 2,91 € pro Wohnung und Monat

Das ist extrem günstig.

Ein Business-Anschluss mit vergleichbarer Leistung könnte deutlich teurer werden, bietet dafür aber oft klarere Regelungen und Support.

------------------------------------------------------------------------

## Ein Punkt, den ich bei euch besonders interessant finde

Die Genossenschaft hat einen starken Vorteil:

Sie besitzt die passive Infrastruktur.

Wenn ihr einmal sauber baut:

    Telekom-Übergabe
          |
       Router
          |
       Switch
          |
      Hausverkabelung
          |
      Wohnungen

dann gehört dieses Netz der Genossenschaft und ist unabhängig vom einzelnen Mieter.

Das ist eigentlich genau das Szenario, für das man eine solche Struktur bauen möchte.

------------------------------------------------------------------------

## Was ich vor Vertragsabschluss klären würde

Bei Telekom konkret schriftlich bestätigen lassen:

1.  **Nutzung durch 24 Mietparteien zulässig?**
2.  **Bekommen FTTH-Anschlüsse ein delegiertes IPv6-Präfix (/56)?**
3.  **Ist ein eigener Router hinter dem ONT möglich?**
4.  **Ist ein Bridge-/Modem-Betrieb möglich?**
5.  **Gibt es Einschränkungen wegen Telefonie?**

Der wichtigste Punkt für euer Konzept ist Nr. 2: Ihr wollt nicht nur Internet, sondern ein sauberes IPv6-Design.

------------------------------------------------------------------------

Nebenbei: Für eine Genossenschaft würde ich auch prüfen, ob ein **Geschäftskunden- oder Immobilienprodukt der Telekom** existiert. Nicht unbedingt wegen der Bandbreite, sondern wegen der Vertragsklarheit. Die Geschäftskunden-Leistungsbeschreibungen führen ebenfalls Glasfaser-1.000-Produkte mit 1.000/500 Mbit/s auf. ([Telekom](https://www.telekom.de/agb/downloads/55609.pdf?utm_source=chatgpt.com))

Technisch ist euer Konzept jedenfalls sehr sauber. Die einzige offene Baustelle ist wirklich die **vertragliche Einordnung des Anschlusses**.

------------------------------------------------------------------------

**Sources:**

- [Glasfaser-Anschluss: Stabiles Internet für Ihr Zuhause](https://www.telekom.de/netz/glasfaseranschluss?utm_source=chatgpt.com)

## Prompt:

7/20/2026, 4:42:21 PM

hier noch die AGB zu "Telekom Magenta Zuhause"

steht da was drin zum thema "anschluss teilen" oder "gemeinsame nutzung vom internetanschluss" oder "anschluss überlassen"...?

------------------------------------------------------------------------

<https://www.telekom.de/agb/downloads/55441.pdf>

Stand: 04.02.2026 Seite 1  
Allgemeine Geschäftsbedingungen  
Festnetz- und Mobilfunk-Anschlüsse  
Stand: 04.02.2026  
1 Wer ist Ihr Vertragspartner?  
Ihr Vertragspartner ist die Telekom Deutschland GmbH (im Folgenden als „Telekom“ abgekürzt), Landgrabenweg 149,  
53227 Bonn (Amtsgericht Bonn HRB 5919).  
2 Wie kommt der Vertrag zustande?  
Soweit wir mit Ihnen nichts anderes vereinbart haben, kommt der Vertrag mit Zugang unserer Auftragsbestätigung  
zustande. Falls wir unsere Leistung früher bereitstellen, kommt der Vertrag schon mit Bereitstellung der Leistung  
zustande.  
3 Welche Leistungen erbringt die Telekom?  
3.1 Allgemein  
Unsere Leistungen ergeben sich aus diesen Allgemeinen Geschäftsbedingungen (AGB), den jeweiligen  
Leistungsbeschreibungen und Preislisten.  
3.2 Eingesetzte Netztechnologie und Technik  
Die Nennung der für die Leistungserbringung von uns eingesetzten Netztechnologie und Technik in Vertragsunterlagen  
oder auf Internetseiten erfolgt zur Information und stellt – soweit nicht ausdrücklich als vertragliche Leistung  
vereinbart - keine vertragliche Vereinbarung dieser Netztechnologie und Technik dar. Wir sind in der Wahl der zur  
Erbringung der jeweils vereinbarten Leistungen eingesetzten Netztechnologie und Technik frei. Zur Netztechnologie  
und Technik gehören zum Beispiel Netz- und Übertragungstechnologien und -protokolle, technische Infrastrukturen  
und Plattformen sowie Benutzeroberflächen. Um auf technologische Neuerungen reagieren zu können, sind wir  
berechtigt, jederzeit Änderungen dieser technischen Mittel vorzunehmen, wenn dadurch die vertraglich vereinbarten  
Leistungen nicht verändert werden. Führen Änderungen bei der Netztechnologie und Technik zu Änderungen der  
vertraglich vereinbarten Leistungen, gelten die Regelungen der Ziffer 8 \[Änderungen von AGB und Leistungen\].  
4 Welche Pflichten und Obliegenheiten haben Sie?  
4.1 Auch Sie haben Pflichten und sogenannte Obliegenheiten. Sie sind insbesondere verpflichtet,  
a) die bereitgestellten Leistungen ausschließlich zu den vertraglich vereinbarten Zwecken und im vertraglich  
vereinbarten Umfang zu nutzen,  
b) ein SEPA-Lastschriftmandat zu erteilen (Ausnahme: Tarif Call Start),  
c) Änderungen Ihres Namens, Ihrer Anschrift, der Bankverbindung, des Rechnungsempfängers sowie der für die  
Vertragsabwicklung benannten E-Mail-Adresse unverzüglich der Telekom mitzuteilen,  
d) persönliche Zugangsdaten (wie Passwort/PIN/PUK) geheim zu halten und diese unverzüglich zu ändern, falls Sie  
vermuten, dass unberechtigte Personen die Zugangsdaten kennen,  
e) den Verlust der SIM-Karte/des eSIM-Profils unverzüglich der Telekom mitzuteilen. Sie müssen nur die Entgelte  
zahlen, die angefallen sind, bis Sie uns benachrichtigt haben,  
f) falls Sie sich für Erwachsenenangebote über ein Altersverifikationssystem angemeldet haben, sicherzustellen,  
dass diese Angebote Minderjährigen nicht zugänglich sind,  
g) Schutzrechte und Persönlichkeitsrechte Dritter (z.B. Urheber- und Markenrechte) zu beachten.  
Allgemeine Geschäftsbedingungen  
Festnetz- und Mobilfunk-Anschlüsse  
Stand: 04.02.2026 Seite 2  
4.2 Bei Festnetz-Anschlüssen bzw. Leistungen an festen Standorten sind Sie zudem insbesondere verpflichtet,  
a) soweit wir für die Erbringung der Leistung Zugang zum Grundstück oder den darauf befindlichen Gebäuden  
benötigen, uns den Zugang auf Ihre Kosten zu ermöglichen,  
b) den elektrischen Strom sowie die Erdung für die Installation, den Betrieb und die Instandhaltung auf Ihre Kosten  
bereitzustellen,  
c) Instandhaltungs- und Änderungsarbeiten am Anschluss nur von der Telekom ausführen zu lassen,  
d) Softwareänderungen der Telekom am Media-Receiver zuzulassen (z. B. automatische Updates).  
e) einen Glasfaseranschluss nach Mitteilung durch die Telekom unverzüglich zu aktivieren.  
5 Was ist nicht erlaubt?  
5.1 Ihnen ist es nicht gestattet,  
a) die Leistungen ohne unsere Zustimmung Dritten zum alleinigen Gebrauch zu überlassen,  
b) selbst als Anbieter von Telekommunikationsdiensten mittels der von der Telekom überlassenen Leistungen  
aufzutreten,  
c) die Inhalte der Leistung MagentaTV (insbesondere TV- und Videoinhalte) für gewerbliche Zwecke zu verwenden  
oder öffentlich wiederzugeben (z. B. in Gaststätten, Hotels oder Krankenhäusern),  
d) bei Festnetz-Anschlüssen bzw. Leistungen an festen Standorten eine zweite Verbindung mittels Point to Point  
Protocol over Ethernet (PPPoE) aufzubauen.  
5.2 Ihnen ist es nicht gestattet, pauschal abgegoltene Leistungen (z. B. Flatrates) wie folgt zu nutzen:  
a) für das Angebot von Mehrwertdiensten sowie das Angebot oder die Nutzung von  
Massenkommunikationsdiensten (insbesondere Faxbroadcastdienste, Call-Center-, Telefonmarketing- und  
Marktforschungsdienstleistungen),  
b) für die dauerhafte oder zyklische Vernetzung oder Verbindung von Standorten bzw. Telekommunikationsanlagen,  
c) für Machine-to-Machine-(M2M)-Anwendungen bzw. -verbindungen,  
d) für den Einsatz in Vermittlungs- und Übertragungssystemen, die dazu dienen Sprach- oder Datenverbindungen  
eines Dritten an einen anderen Dritten ein- oder weiterzuleiten (z.B. SIM-Boxing),  
e) für Verbindungen, die dem Zweck dienen, dass Sie oder ein Dritter aufgrund der Verbindung und/oder aufgrund  
der Verbindungsdauer Auszahlungen oder andere Gegenleistungen erhalten sollen (z. B. Gegenleistungen für  
Anrufe zu Chatlines oder Werbehotlines),  
f) für Verbindungen, die nicht der direkten Kommunikation zu einem anderen Teilnehmer dienen, sondern nur dem  
Zweck des Verbindungsaufbaus und/oder der Verbindungsdauer,  
g) für Verbindungen, die mittels automatisierter Verfahren (z. B. ausführbare Routinen, Apps, Programme)  
hergestellt werden,  
h) im Falle von pauschal abgegoltenen Telefonie- und/oder Telefaxverbindungen: Verbindungen herzustellen  
▪ die der Dateneinwahl dienen und mit denen Sie Zugang zum Internet erhalten,  
▪ deren Leistungen über die direkte Kommunikationsverbindung per Telefon und/oder Fax zu einem anderen  
Teilnehmer hinausgehen (z. B. Mehrwertdienste mit geografischer Festnetzrufnummer als Einwahlrufnummer.  
Das sind z.B. Services für Chat, Call Through, Call by Call, Call Back, Internet by Call),  
▪ die dauerhaft umgeleitet werden (z.B. für Überwachungs- und Kontrollfunktionen).  
Allgemeine Geschäftsbedingungen  
Festnetz- und Mobilfunk-Anschlüsse  
Stand: 04.02.2026 Seite 3  
5.3 Rechtswidrige Nutzung  
Ihnen ist es nicht gestattet unsere Leistungen für die Übermittlung oder Verbreitung oder den Hinweis auf rechts- oder  
sittenwidrige Inhalte zu nutzen. Sie dürfen ferner keine gesetzlich verbotenen, unaufgeforderten Informationen, Sachen  
und sonstigen Leistungen (wie z. B. unerwünschte und unverlangte Werbung) übersenden.  
5.4 Weitere Regelungen  
Weitere unzulässige Nutzungen für einzelne Produkte können in den jeweiligen Leistungsbeschreibungen und  
Preislisten geregelt sein.  
6 Wie rechnen wir ab und wann müssen Sie bezahlen?  
6.1 Fälligkeit der Preise  
Die Preise werden mit Zugang der Rechnung fällig.  
6.2 Abrechnung der Leistungen  
a) Wir sind berechtigt, die monatlichen Preise im Voraus in Rechnung zu stellen.  
b) Sonstige Preise, insbesondere Verbindungspreise, zahlen Sie nach Erbringung der Leistung.  
6.3 Zahlungsart und -frist  
Die Zahlung des Rechnungsbetrages kann ausschließlich per Einzug über ein SEPA-Lastschriftmandat erfolgen. Wir  
buchen den Betrag frühestens fünf Werktage nach Zugang der Rechnung und der Ankündigung der Abbuchung vom  
vereinbarten Konto ab. Ein Rechnungsbetrag muss spätestens am 10 Tag nach Zugang der Rechnung auf dem in der  
Rechnung angegebenen Konto der Telekom gutgeschrieben sein.  
6.4 Beanstandungen  
Sind Sie mit der Rechnung nicht einverstanden, müssen Ihre Beanstandungen innerhalb von acht Wochen ab  
Rechnungszugang bei der Telekom eingegangen sein. Wir werden Sie in den Rechnungen auf die Folgen einer  
unterlassenen rechtzeitigen Beanstandung besonders hinweisen. Ihre gesetzlichen Ansprüche bei Beanstandungen  
nach Fristablauf bleiben unberührt.  
6.5 Zurückbehaltungsrecht  
Ihnen steht die Geltendmachung eines Zurückbehaltungsrechts nur wegen Gegenansprüchen aus diesem Vertrag zu.  
7 Was ist, wenn Sie nicht oder nicht rechtzeitig zahlen?  
Bei Zahlungsverzug mit einem Betrag von mindestens einhundert Euro können wir die zu erbringende Leistung auf Ihre  
Kosten und unter den Voraussetzungen des § 61 Telekommunikationsgesetz (TKG) sperren. Sie bleiben in diesem Fall  
verpflichtet, die monatlichen Preise zu zahlen. Die Geltendmachung weiterer Ansprüche wegen Zahlungsverzuges  
bleibt unberührt.  
8 Wie können wir unsere AGB und Leistungen ändern?  
8.1 Änderung der AGB  
Die AGB können geändert werden, soweit dies zur Anpassung an Entwicklungen erforderlich ist, die bei Vertragsschluss  
nicht vorhersehbar waren und die die Telekom nicht veranlasst hat oder beeinflussen kann und deren  
Nichtberücksichtigung die Ausgewogenheit des Vertrages in nicht unbedeutendem Maße stören würde und hierdurch  
wesentliche Regelungen des Vertrages nicht geändert werden. Wesentliche Regelungen sind insbesondere solche über  
die Laufzeit und die Kündigung des Vertrages.  
Allgemeine Geschäftsbedingungen  
Festnetz- und Mobilfunk-Anschlüsse  
Stand: 04.02.2026 Seite 4  
Die AGB können auch angepasst werden, soweit damit nach Vertragsschluss entstandene Regelungslücken  
geschlossen werden, die nicht unerhebliche Schwierigkeiten bei der Durchführung des Vertrages verursachen. Dies  
kann insbesondere der Fall sein, wenn sich die Rechtsprechung zur Wirksamkeit von Bestimmungen dieser AGB ändert,  
wenn eine oder mehrere Bestimmungen dieser AGB von der Rechtsprechung für unwirksam erklärt werden oder eine  
Gesetzesänderung zur Unwirksamkeit einer oder mehrerer Bestimmungen dieser AGB führt.  
8.2 Änderung der Leistungen  
Die vertraglich vereinbarten Leistungen können geändert werden, wenn dies aus triftigem Grund erforderlich ist und  
die Änderung für Sie zumutbar ist. Ein triftiger Grund liegt vor, wenn die Änderung zur Anpassung an technische  
Entwicklungen (z.B. Änderung von technischen Plattformen, Systemen, Netztechnologien) oder aufgrund gesetzlicher  
oder behördlicher Vorgaben erforderlich ist und dies bei Vertragsschluss nicht vorhersehbar war. Eine Änderung ist  
zumutbar, wenn das Gleichgewicht von Leistung und Gegenleistung nicht oder nur ganz unerheblich beeinträchtigt  
wird und unser Änderungsinteresse höher wiegt als Ihr Interesse an einer unveränderten Beibehaltung der Leistung.  
8.3 Mitteilung der Änderungen und Kündigungsrecht  
Änderungen der AGB oder der Leistungen gemäß Ziffer 8.1 bis 8.2 werden wir Ihnen mindestens einen Monat,  
höchstens zwei Monate vor ihrem geplanten Wirksamwerden in Textform mitteilen. Ihnen steht bei Änderungen das  
Recht zu, den Vertrag ohne Einhaltung einer Kündigungsfrist und ohne Kosten frühestens zum Zeitpunkt des  
Wirksamwerdens der Änderungen in Textform zu kündigen. Sie können die Kündigung innerhalb von drei Monaten ab  
Zugang der Änderungsmitteilung erklären. Auf den Inhalt und den Zeitpunkt der Vertragsänderung und ein  
bestehendes Kündigungsrecht werden wir Sie in der Änderungsmitteilung besonders hinweisen.  
Ein Kündigungsrecht steht Ihnen nicht zu, wenn die Änderungen  
▪ ausschließlich zu Ihrem Vorteil sind,  
▪ rein administrativer Art sind und keine negativen Auswirkungen auf Sie haben oder  
▪ unmittelbar durch Unionsrecht oder innerstaatlich geltendes Recht vorgeschrieben sind.  
9 Wie können wir unsere Preise ändern?  
9.1 Preisanpassung  
Wir sind berechtigt, die auf der Grundlage dieses Vertrages zu zahlenden Preise der Entwicklung der Gesamtkosten  
anzupassen, die für die Berechnung des vereinbarten Preises maßgeblich sind. Die Anpassung erfolgt nach billigem  
Ermessen auf Basis von § 315 Bürgerliches Gesetzbuch (BGB).  
a) Die Gesamtkosten bestehen insbesondere aus Kosten für Netzbereitstellung, Netznutzung und Netzbetrieb (z. B.  
für Technik, besondere Netzzugänge und Netzzusammenschaltungen, technischer Service), Kosten für die  
Kundenbetreuung (z. B. für Service-Hotlines, Abrechnungs- und IT-Systeme), Personal- und  
Dienstleistungskosten, Kosten für Vorleistungen Dritter, Energiekosten, Gemeinkosten (z. B. für Verwaltung,  
Marketing, Mieten, Zinsen) sowie hoheitlich auferlegten Gebühren, Aus-lagen und Beiträgen (z. B. aus §§ 223,  
224 TKG).  
b) Eine Preiserhöhung kommt in Betracht und eine Preissenkung ist vorzunehmen, wenn sich die Gesamtkosten  
erhöhen oder absenken.  
c) Steigerungen bei einer Kostenart, z. B. Kosten für die Netznutzung, dürfen nur in dem Umfang für eine  
Preiserhöhung herangezogen werden, in dem kein Ausgleich durch rückläufige Kosten in anderen Bereichen etwa  
bei der Kundenbetreuung erfolgt. Bei Kostensenkungen sind von der Telekom die Preise zu senken, soweit diese  
Kostensenkungen nicht durch Steigerungen bei einer anderen Kostenart ausgeglichen werden. Wir werden bei der  
Ausübung des billigen Ermessens die jeweiligen Zeitpunkte einer Preisänderung so wählen, dass  
Allgemeine Geschäftsbedingungen  
Festnetz- und Mobilfunk-Anschlüsse  
Stand: 04.02.2026 Seite 5  
Kostensenkungen nicht nach für Sie ungünstigeren Maßstäben berechnet werden als Kostenerhöhungen, also  
Kostensenkungen mindestens in gleichem Umfang preiswirksam werden wie Kostenerhöhungen.  
9.2 Mitteilung der Preisanpassung und Kündigungsrecht  
Änderungen der Preise nach Ziffer 9.1 werden wir Ihnen mindestens einen Monat, höchstens zwei Monate vor ihrem  
geplanten Wirksamwerden in Textform mitteilen. Ihnen steht bei einer Preiserhöhung das Recht zu, den Vertrag ohne  
Einhaltung einer Kündigungsfrist und ohne Kosten frühestens zum Zeitpunkt des Wirksamwerdens der Preisanpassung  
in Textform (z. B. per Brief oder E-Mail) zu kündigen. Sie können die Kündigung innerhalb von drei Monaten ab Zugang  
der Änderungsmitteilung erklären. Auf den Inhalt und den Zeitpunkt der Vertragsänderung und ein bestehendes  
Kündigungsrecht auf Grund der Änderung werden wir Sie in der Änderungsmitteilung besonders hinweisen. Im Übrigen  
bleibt § 315 BGB unberührt. Ein Kündigungsrecht steht Ihnen nicht zu, wenn die Preiserhöhung unmittelbar durch  
Unionsrecht oder innerstaatlich geltendes Recht vorgeschrieben sind.  
9.3 Preisanpassung aufgrund Änderung der Umsatzsteuer  
Unabhängig von den Regelungen der Ziffer 9.1 bis 9.2 sind wir für den Fall einer Erhöhung der gesetzlichen  
Umsatzsteuer berechtigt und für den Fall einer Senkung verpflichtet, die Preise zum Zeitpunkt der jeweiligen Änderung  
entsprechend anzupassen. Bei dieser Preisanpassung haben Sie kein Kündigungsrecht.  
10 Wie haften wir?  
Wir haften nach § 70 TKG und dem Produkthaftungsgesetz. Außerhalb des Anwendungsbereichs dieser Regelungen  
gilt Folgendes:  
a) Wir haften bei Vorsatz oder grober Fahrlässigkeit sowie bei Fehlen einer garantierten Eigenschaft für alle darauf  
zurückzuführende Schäden unbeschränkt.  
b) Bei leichter Fahrlässigkeit haften wir im Fall der Verletzung des Lebens, des Körpers oder der Gesundheit  
unbeschränkt.  
Wenn wir durch leichte Fahrlässigkeit mit unserer Leistung in Verzug geraten sind, unsere Leistung unmöglich  
geworden ist oder wir eine wesentliche Pflicht verletzt haben, ist die Haftung für darauf zurückzuführende Sachund Vermögensschäden auf den vertragstypisch vorhersehbaren Schaden begrenzt. Eine wesentliche Pflicht ist  
eine Pflicht, deren Erfüllung die ordnungsgemäße Durchführung des Vertrages überhaupt erst ermöglicht, deren  
Verletzung die Erreichung des Vertragszweckes gefährdet und auf deren Einhaltung Sie regelmäßig vertrauen  
dürfen.  
c) Für den Verlust von Daten haften wir bei leichter Fährlässigkeit unter den Voraussetzungen und im Umfang von  
Ziffer 10 b) nur, soweit Sie Ihre Daten regelmäßig so gesichert haben, dass diese mit vertretbarem Aufwand  
wiederhergestellt werden können.  
d) Die Haftung für alle übrigen Schäden ist ausgeschlossen.  
11 Was gilt hinsichtlich Vertragslaufzeit und Kündigung?  
11.1 Vertragslaufzeit und Kündigungsfrist  
Die Mindestvertragslaufzeiten, Kündigungsfristen und eventuelle Verlängerungsfristen sind in der Preisliste des  
jeweiligen Produkts oder Tarifs oder der jeweiligen Zubuchoption geregelt. Mit Kündigung des Vertrages enden auch  
Vertragsverhältnisse über Zubuchoptionen.  
11.2 Außerordentliche Kündigung  
Das Recht aus wichtigem Grund zu kündigen, bleibt unberührt.  
Allgemeine Geschäftsbedingungen  
Festnetz- und Mobilfunk-Anschlüsse  
Stand: 04.02.2026 Seite 6  
11.3 Form der Kündigung  
Die Kündigung muss in Textform erfolgen.  
11.4 Pauschalisierter Schadensersatz  
Kündigen wir den Vertrag oder eine Zubuchoption vorzeitig aus einem von Ihnen zu vertretenden wichtigen Grund, sind  
Sie verpflichtet, uns einen pauschalierten Schadensersatz zu zahlen. Der in einer Summe zu zahlende Betrag beläuft  
sich auf die Hälfte der bis zum Ablauf der vereinbarten Vertragslaufzeit zu zahlenden monatlichen Preise. Der  
Schadensbetrag ist höher oder niedriger anzusetzen, wenn wir einen wesentlich höheren Schaden nachweisen oder Sie  
nachweisen, dass ein wesentlich geringerer oder gar kein Schaden entstanden ist.  
12 Welche Informationen sind noch wichtig für Sie?  
12.1 Messverfahren  
Informationen über die von der Telekom zur Messung und Kontrolle des Datenverkehrs eingerichteten Verfahren, die  
eine Kapazitätsauslastung oder Überlastung einer Netzwerkverbindung vermeiden sollen, und Informationen über die  
möglichen Auswirkungen finden Sie im Internet unter [www.telekom.de/messverfahren](http://www.telekom.de/messverfahren).  
12.2 Kontaktdaten  
Die Kontaktadressen für Serviceleistungen finden Sie im Internet unter [www.telekom.de/kontakt](http://www.telekom.de/kontakt).  
12.3 Preisverzeichnis  
Ein allgemein zugängliches, vollständiges und gültiges Preisverzeichnis finden Sie unter [www.telekom.de/agb](http://www.telekom.de/agb).  
12.4 Dienstesicherheit  
Eine Auflistung der Maßnahmen, mit denen die Telekom auf Sicherheits- oder Integritätsverletzungen oder auf  
Bedrohungen oder Schwachstellen reagieren kann, finden Sie im Internet unter [www.telekom.de/dienstesicherheit](http://www.telekom.de/dienstesicherheit).  
12.5 Voraussetzung für Anbieterwechsel und Rufnummernmitnahme  
Damit im Falle eines Anbieterwechsels bzw. der Rufnummernmitnahme die Leistung nicht oder nicht länger als einen  
Arbeitstagunterbrochen wird, müssen folgende Voraussetzungen erfüllt sein:  
a) Anbieterwechsel Festnetz und Mobilfunk zum Vertragsende  
Sie müssen Ihren Vertrag mit der Telekom fristgerecht gekündigt haben. Der vom neuen Anbieter übermittelte und  
vollständig ausgefüllte Anbieterwechselauftrag muss bei Festnetzanschlüssen spätestens sieben Arbeitstage, bei  
Mobilfunkanschlüssen spätestens acht Arbeitstage vor dem Datum des Vertragsendes bei der Telekom eingehen.  
b) Rufnummernmitnahme Mobilfunk vor Vertragsende  
Sie können Ihre Rufnummer jederzeit auf einen anderen Mobilfunk-Anbieter übertragen. Der vom anderen Anbieter  
übermittelte Portierungsauftrag muss mit den vollständig ausgefüllten Angaben spätestens acht Arbeitstage vor dem  
Datum der Rufnummernübertragung der Telekom zugehen. Der Mobilfunkvertrag mit der Telekom bleibt von der  
Rufnummernübertragung ansonsten unberührt.  
12.6 Entschädigung bei Anbieterwechsel  
Wird bei einem Anbieterwechsel Ihr Anschluss länger als einen Arbeitstag unterbrochen, können Sie von uns, sofern wir  
abgebender Anbieter sind, für jeden weiteren Arbeitstag der Unterbrechung eine Entschädigung in Höhe von 10 Euro  
oder 20 Prozent des vertraglich vereinbarten Monatsentgeltes, je nachdem welcher Betrag höher ist, verlangen. Dies  
gilt nicht, wenn Sie die Verzögerung zu vertreten haben. Die vorgennannte Entschädigung steht Ihnen einmalig auch  
dann zu, wenn wir als aufnehmender oder abgebender Anbieter einen mit Ihnen vereinbarten Kundendienst- oder  
Allgemeine Geschäftsbedingungen  
Festnetz- und Mobilfunk-Anschlüsse  
Stand: 04.02.2026 Seite 7  
Installationstermin versäumen. Die Entschädigungszahlungen stehen Ihnen nicht zu, wenn Sie die Verzögerung oder  
das Versäumnis des Termines zu vertreten haben.  
12.7 Entschädigung bei Rufnummernmitnahme  
Erfolgt eine Rufnummernmitnahme und technische Aktivierung der Rufnummer nicht spätestens innerhalb des  
Arbeitstages, der auf den mit Ihnen vereinbarten Tag folgt, steht Ihnen für jeden Tag der Verzögerung eine  
Entschädigung in Höhe von 10 Euro zu, sofern wir als aufnehmender oder abgebender Anbieter die Verzögerung zu  
vertreten haben.  
12.8 Entschädigung bei Kundendienst- oder Installationstermin  
Halten wir einen vereinbarten Kundendienst- oder Installationstermin im Falle einer Entstörung, eines Umzuges oder  
eines Anbieterwechsels nicht ein, steht Ihnen eine Entschädigung in Höhe von 10 Euro oder 20 Prozent des vertraglich  
vereinbarten Monatsentgeltes, je nachdem welcher Betrag höher ist, zu. Dies gilt nicht, wenn Sie die die Versäumnis  
des Termines, zu vertreten haben.  
12.9 Abweichungen bei der Geschwindigkeit oder bei anderen Dienstqualitätsparametern  
Im Falle von  
a) erheblichen, kontinuierlichen oder regelmäßig wiederkehrenden Abweichungen bei der Geschwindigkeit oder bei  
anderen Dienstqualitätsparametern zwischen der tatsächlichen Leistung des Internetzugangsdienstes und der  
gemäß den des Artikels 4 Abs. 1 Buchstabe a bis d der EU-Verordnung 2015/2120 angegebenen Leistung, die  
durch einen von der Bundesnetzagentur bereitgestellten oder von ihr oder einem von ihr beauftragten Dritten  
zertifizierten Überwachungsmechanismus ermittelt wurden, oder  
b) anhaltenden oder häufig auftretenden erheblichen Abweichungen zwischen der tatsächlichen und der im Vertrag  
angegebenen Leistung eines Telekommunikationsdienstes mit Ausnahme eines Internetzugangsdienstes  
sind Sie, sofern Sie Verbraucher sind, unbeschadet sonstiger Rechtsbehelfe berechtigt, das vertraglich vereinbarte  
Entgelt unter den Voraussetzungen und im Umfang des § 57 Abs. 4 TKG zu mindern oder den Vertrag außerordentlich  
ohne Einhaltung einer Kündigungsfrist zu kündigen. Im Falle der außerordentlichen Kündigung gilt § 56 Abs. 4 Satz 2  
bis 4 TKG.  
12.10 Netzseitige Sperre für bestimmte Rufnummernbereiche  
Sie können verlangen, dass die Nutzung Ihres Netzzuganges für bestimmte Rufnummernbereiche unentgeltlich durch  
die Telekom netzseitig gesperrt wird, soweit dies technisch möglich ist.  
12.11 Netzseitige Sperre der Identifizierung des Mobilfunk Anschlusses  
Sie können verlangen, dass die Identifizierung Ihres Mobilfunk-Anschlusses zur Inanspruchnahme und Abrechnung  
einer neben der Verbindung erbrachten Leistung unentgeltlich durch die Telekom netzseitig gesperrt wird.  
12.12 Eintrag in Endnutzerverzeichnisse  
Sie können jederzeit verlangen, mit Ihrer Rufnummer, Ihrem Namen, Ihrem Vornamen und Ihrer Anschrift in ein  
allgemein zugängliches Endnutzerverzeichnis unentgeltlich eingetragen oder gespeichert zu werden oder diese  
Angaben berichtigen oder wieder löschen zu lassen.  
12.13 Streitbeilegung durch Schlichtungsverfahren  
Zur Beilegung eines Streits mit der Telekom über die in § 68 TKG genannten Fälle können Sie durch einen Antrag ein  
Schlichtungsverfahren bei der Verbraucherschlichtungsstelle Telekommunikation der Bundesnetzagentur in Bonn  
einleiten. Die Telekom ist bereit an Schlichtungsverfahren vor der Bundesnetzagentur teilzunehmen. An  
Streitbeilegungsverfahren vor anderen Verbraucherschlichtungsstellen nehmen wir nicht teil.  
Allgemeine Geschäftsbedingungen  
Festnetz- und Mobilfunk-Anschlüsse  
Stand: 04.02.2026 Seite 8  
12.14 Personenbezogene Daten des Kunden vor Freischaltung  
Vor Freischaltung einer Telekommunikationsdienstleistung im Sinne des § 172 TKG sind als personenbezogene Daten  
von Ihnen bereitzustellen und von uns zu erfassen: Namen und die Anschrift des Anschlussinhabers, bei natürlichen  
Personen deren Geburtsdatum und bei Festnetzanschlüssen die Anschrift des Anschlusses.  
13 Was gilt sonst noch?  
13.1 Leistungserbringung durch Dritte  
Wir sind berechtigt, die Leistungen durch Dritte zu erbringen. Wir haften für die Leistungserbringung durch Dritte wie  
für eigenes Handeln.  
13.2 Übertragung von Rechten und Pflichten auf Dritte  
Die Telekom ist berechtigt, die sich aus diesem Vertrag ergebenden Rechte und Pflichten ohne Ihre Zustimmung auf die  
Deutsche Telekom AG, Friedrich-Ebert-Allee 140, 53113 Bonn (Amtsgericht Bonn HRB 6794) oder auf einen sonstigen  
Dritten zu übertragen. Ihnen steht für den Fall der Übertragung auf einen namentlich nicht genannten Dritten das Recht  
zu, den Vertrag mit der Telekom ohne Einhaltung einer Frist zu kündigen.  
13.3 Gerichtsstand  
Sind Sie Kaufmann, juristische Person des öffentlichen Rechts oder ein öffentlich-rechtliches Sondervermögen, gilt für  
die vertraglichen Beziehungen deutsches Recht und für alle Streitigkeiten aus oder im Zusammenhang mit diesem  
Vertrag ist Gerichtsstand Bonn. Ein etwaiger ausschließlicher Gerichtsstand ist vorrangig.  
[www.telekom.de](http://www.telekom.de)

## Response:

7/20/2026, 4:42:21 PM

Danke für das vollständige Dokument. Die relevante Stelle ist tatsächlich vorhanden – und sie ist für euren Fall der Knackpunkt.

Die entscheidenden Klauseln sind:

> **5.1 a)**  
> „Ihnen ist es nicht gestattet, die Leistungen ohne unsere Zustimmung Dritten zum alleinigen Gebrauch zu überlassen“

und

> **5.1 b)**  
> „Ihnen ist es nicht gestattet, selbst als Anbieter von Telekommunikationsdiensten mittels der von der Telekom überlassenen Leistungen aufzutreten“

------------------------------------------------------------------------

## Was bedeutet das für eure Genossenschaft?

### 5.1 a) "Dritten zum alleinigen Gebrauch überlassen"

Das zielt auf Fälle wie:

- Firma A kauft Telekom-Anschluss
- Firma A gibt ihn komplett an Firma B weiter
- Firma A nutzt ihn selbst nicht mehr

Beispiel:

    Telekom
       |
     Firma A
       |
     Firma B (alleiniger Nutzer)

Das wäre eine unerlaubte Überlassung.

Euer Fall ist anders:

    Telekom
       |
    Genossenschaft
       |
    24 Wohnungen

Die Genossenschaft nutzt den Anschluss als Infrastruktur für das Gebäude. Die Wohnungen sind keine separaten "Kunden", denen der Anschluss vollständig überlassen wird.

Allerdings: Die Klausel ist bewusst allgemein formuliert. Ob Telekom das genauso sieht, sollte man schriftlich klären.

------------------------------------------------------------------------

## 5.1 b) "selbst als Anbieter von Telekommunikationsdiensten auftreten"

Das ist die interessantere Klausel.

Ein klassischer ISP wäre:

    Telekom
       |
     Genossenschaft
       |
     Kunde A
     Kunde B
     Kunde C

mit:

- eigenen Internetverträgen
- eigener Abrechnung
- Werbung
- Kundenbeziehung
- Angebot am Markt

Das wäre ziemlich klar ein Anbieter.

Euer Modell:

    Telekom
       |
    Genossenschaft
       |
    Mieter (Bestandteil Wohnung)

ist deutlich näher an:

- Firmeninternet mit Mitarbeitern
- Hotel mit Internet für Gäste
- Krankenhausnetz
- Studentenwohnheimnetz
- Campusnetz

Die entscheidende Frage ist: **Sind die Mieter "Endnutzer innerhalb einer Infrastruktur" oder "Kunden eines Telekommunikationsdienstes der Genossenschaft"?**

Bei euch spricht vieles für Ersteres.

------------------------------------------------------------------------

## Der Punkt "Bestandteil der Miete" ist wichtig

Deine Aussage:

> der internetanschluss gehört fest zur wohnung, also ist nicht optional

hilft eher.

Denn dann ist die Situation:

> Die Genossenschaft vermietet Wohnungen mit einer technischen Ausstattung, zu der ein Internetzugang gehört.

Nicht:

> Die Genossenschaft verkauft Internetanschlüsse.

Das ist ein großer Unterschied.

------------------------------------------------------------------------

## Aber: Ich würde nicht alleine aus den AGB ableiten "alles erlaubt"

Die AGB geben Telekom ein Argument, wenn sie sagen würde:

> "Ihr habt 24 eigenständige Haushalte angeschlossen, das ist faktisch eine Weitergabe."

Ob sie das tun würde, ist eine andere Frage.

------------------------------------------------------------------------

## Was ich an eurer Stelle tun würde

Ich würde Telekom nicht fragen:

> "Dürfen wir ISP spielen?"

sondern:

> "Die Wohnungsgenossenschaft errichtet ein internes Gebäudenetz. Ein Telekom-Glasfaseranschluss wird ausschließlich innerhalb des eigenen Gebäudes für die eigenen Mietwohnungen genutzt. Es erfolgt kein öffentliches Angebot, keine Weitervermietung von Internetdiensten und keine Nutzung durch externe Dritte. Ist der Tarif Telekom Glasfaser 1000 hierfür geeignet?"

Das beschreibt euren tatsächlichen Fall.

------------------------------------------------------------------------

## Noch eine technische Vertragsfrage: IPv6

Aus dem AGB-Dokument geht übrigens **nicht** hervor:

- welches IPv6-Präfix geliefert wird
- ob /56 garantiert wird
- ob Prefix Delegation möglich ist
- ob ein eigener Router hinter dem ONT erlaubt ist

Das steht eher in Leistungsbeschreibung/Technikunterlagen.

Für euer Projekt wäre die wichtigste technische Vertragsfrage:

> "Wird bei Telekom Glasfaser 1000 ein delegiertes IPv6-Präfix größer als /64 bereitgestellt (idealerweise /56 via DHCPv6-PD)?"

Ohne das könnt ihr zwar Internet machen, aber nicht sauber 24 IPv6-Netze verteilen.

------------------------------------------------------------------------

## Meine Einschätzung zu eurem konkreten Fall

Mit:

- Genossenschaft als Anschlussinhaber
- eigener Immobilie
- nur eigene Mieter
- keine öffentliche Nutzung
- keine zusätzliche Abrechnung pro Anschluss
- Infrastruktur gehört der Genossenschaft

würde ich das **nicht als typischen unerlaubten Anschluss-Weiterverkauf** einordnen.

Aber: Für 24 Wohnungen würde ich mir von Telekom eine kurze schriftliche Bestätigung holen. Nicht wegen der Technik – die ist einfach –, sondern damit später niemand behauptet, der Anschluss sei vertragswidrig genutzt worden.

## Prompt:

7/20/2026, 4:49:04 PM

ok, also ein "Telekom Business Glasfaser 1000 Pro" tarif für 100 euro pro monat (statt 70 euro pro monat) wäre auch interessant, weil der bietet eine statische IP adresse (das macht DNS einfacher)

<https://geschaeftskunden.telekom.de/business/produkte/internet-festnetz/tarife/business-glasfaser-tarife>

und das wäre immer noch viel billiger als 24 anschlüsse für 24 wohnungen (wovon dann im durchschnitt nur 5% der bandbreite genutzt wird...)

## Response:

7/20/2026, 4:49:04 PM

Ja, genau - unter euren Randbedingungen wird ein **Business Glasfaser Pro**-Tarif sehr interessant. Nicht primär wegen der Bandbreite, sondern wegen der **vertraglichen Klarheit und der festen IP-Adresse**.

Die Telekom beschreibt bei den Business-Glasfaser-Pro-Tarifen, dass diese eine **feste IP-Adresse** enthalten. ([geschaeftskunden.telekom.de](https://geschaeftskunden.telekom.de/business/produkte/internet-festnetz/tarife/business-glasfaser-tarife?utm_source=chatgpt.com)) Eine feste IP ist insbesondere für dauerhaft erreichbare Dienste, VPNs oder feste DNS-Zuordnungen gedacht. ([geschaeftskunden.telekom.de](https://geschaeftskunden.telekom.de/business/hilfe-und-service/geraete-konfigurationen/internet-festnetz/feste-ip-adresse-einrichten?utm_source=chatgpt.com))

Für euer Genossenschaftsmodell sehe ich mehrere Vorteile:

------------------------------------------------------------------------

## 1. Die feste IP löst das DNS-Thema

Bei einem normalen Privatanschluss:

    Genossenschaft
        |
    dynamische IPv4
        |
    DNS via DynDNS

müsstest du mit Änderungen rechnen.

Mit Business:

    Genossenschaft
        |
    203.0.113.10
        |
    DNS A-Record

also:

    router.genossenschaft.de → 203.0.113.10

Das ist sauberer.

------------------------------------------------------------------------

## 2. Für IPv6 ist die feste IPv4 aber weniger wichtig

Euer eigentliches Design ist ja:

    Wohnung 1:
    2001:db8:1234:101::/64

    Wohnung 2:
    2001:db8:1234:102::/64

    ...

    Wohnung 24:
    2001:db8:1234:124::/64

Die Identifikation passiert über IPv6-Präfixe.

Die zentrale öffentliche IPv4-Adresse wäre eher für:

- alte IPv4-only-Dienste
- VPN-Zugang
- Management
- Reverse Proxy
- Monitoring

interessant.

------------------------------------------------------------------------

## 3. Preislich ist das extrem attraktiv

Deine Rechnung ist richtig:

Business:

    100 €/Monat

geteilt durch 24 Wohnungen:

    ≈ 4,17 €/Wohnung/Monat

Dagegen:

24 einzelne Anschlüsse:

    24 × 70 €
    = 1680 €/Monat

Das wäre völlig unwirtschaftlich.

Und auch technisch unsinnig:

- 24 Router
- 24 Glasfaser-ONTs
- 24 Providerverträge
- 24 unterschiedliche IPv6-Präfixe
- keine zentrale Kontrolle

------------------------------------------------------------------------

## 4. Der Business-Tarif passt auch besser zum Betreiber-Modell

Der größte Vorteil ist gar nicht die IP-Adresse, sondern das Signal:

"Dieser Anschluss gehört einer Organisation."

Bei einem Wohnblock mit Genossenschaft ist das viel plausibler als:

"Eine Privatperson hat einen MagentaZuhause-Anschluss und verteilt ihn an 23 Nachbarn."

Der Business-Vertrag passt organisatorisch besser.

------------------------------------------------------------------------

## 5. Ein möglicher Zielaufbau

Ich würde es ungefähr so bauen:

    Telekom Business Glasfaser Pro
                 |
                ONT
                 |
           Router/Firewall
           (OPNsense/MikroTik)
                 |
           Managed Switch
                 |
       +---------+---------+
       |         |         |
    Whg 1     Whg 2    Whg 24

    IPv6:
    /64       /64       /64

Zusätzlich:

    Management-VLAN
        |
    Router
        |
    Switch

damit ihr später nicht physisch an den Switch müsst.

------------------------------------------------------------------------

## Eine Sache würde ich noch prüfen

Bei Business-Tarifen würde ich vor Bestellung gezielt fragen:

1.  **Welches IPv6-Präfix wird delegiert?**

    - /56?
    - /48?
    - nur /64?

2.  **Kann ein eigener Router verwendet werden?**

3.  **Wird DHCPv6 Prefix Delegation unterstützt?**

4.  **Welche feste IP ist gemeint?**

    - feste IPv4?
    - feste IPv6?
    - beides?

Denn für euer Hausnetz ist der wichtigste Punkt nicht die eine feste IP, sondern ein ausreichend großer IPv6-Delegationsblock.

------------------------------------------------------------------------

Noch ein Gedanke: Wenn ihr sowieso neu verkabelt und das Gebäude dauerhaft in Genossenschaftshand bleibt, würde ich sogar überlegen, ob ihr zusätzlich ein kleines **eigenes internes AS (Autonomous System)** mit eigener Provider-unabhängiger IPv6-Adressierung irgendwann interessant findet. Für 24 Wohnungen ist das heute noch Overkill, aber Genossenschaften denken oft in Jahrzehnten - und ein eigenes Netz kann dann unabhängig vom Telekom-Anschluss weiterleben. Für den ersten Aufbau würde ich aber klar bei Provider-Prefix-Delegation bleiben.

------------------------------------------------------------------------

**Sources:**

- [Business Glasfaser Internet Tarife \| Telekom Geschäftskunden](https://geschaeftskunden.telekom.de/business/produkte/internet-festnetz/tarife/business-glasfaser-tarife?utm_source=chatgpt.com)

## Prompt:

7/20/2026, 4:51:52 PM

> eigenes internes AS (Autonomous System) mit eigener Provider-unabhängiger IPv6-Adressierung

ja, das klingt interessant, weil es sind ja nicht nur 24 wohnungen.... insgesamt hat die baugenossenschaft mehrere hundert wohnungen, und jeder wohnblock (je 24 wohnungen) kriegt einen eigenen internetanschluss

## Response:

7/20/2026, 4:51:52 PM

Ah, das ändert die Perspektive deutlich. Bei **mehreren hundert Wohnungen über mehrere Gebäude** ist die Frage nach einer eigenen Netzarchitektur tatsächlich interessant. Ihr seid dann nicht mehr nur bei "ein Internetanschluss für ein Haus", sondern baut im Prinzip ein kleines **Genossenschaftsnetz**.

Ein eigenes **Autonomous System (AS)** kann dabei eine sinnvolle langfristige Option sein - aber ich würde es nicht als ersten Schritt sehen. Es gibt eine Zwischenstufe, die vermutlich für euch optimal ist.

------------------------------------------------------------------------

## Variante 1: Jeder Wohnblock bleibt eigenständig (wahrscheinlich der beste Start)

Jeder Block:

``` text
Telekom Business Glasfaser
          |
      Router
          |
    Switch / VLANs
          |
  24 Wohnungen
```

Telekom liefert z. B.:

    Block A:
    2001:db8:aaaa::/56

    Block B:
    2001:db8:bbbb::/56

    Block C:
    2001:db8:cccc::/56

Vorteile:

- einfach
- ausfallsicher
- kein eigenes Routing-Management
- kein BGP
- wenig Personalaufwand

Für einige hundert Wohnungen ist das völlig ausreichend.

------------------------------------------------------------------------

## Variante 2: Genossenschaft bekommt eigenes IPv6-Präfix

Mit einem eigenen Provider-unabhängigen IPv6-Adressbereich (PI = Provider Independent) könnte es so aussehen:

    Genossenschaft
    2001:db8:5000::/48

Dann:

    Block A
    2001:db8:5000:0100::/56

    Block B
    2001:db8:5000:0200::/56

    Block C
    2001:db8:5000:0300::/56

und Wohnungen:

    Wohnung 1:
    2001:db8:5000:0101::/64

    Wohnung 2:
    2001:db8:5000:0102::/64

Der Vorteil:

Der Adressraum gehört **der Genossenschaft**, nicht der Telekom.

------------------------------------------------------------------------

## Aber: Dafür braucht ihr ein AS

Damit das Internet weiß:

> "Dieses Präfix gehört zur Genossenschaft"

braucht ihr:

- ein eigenes Autonomous System (ASN)
- BGP-Verbindungen zu Providern
- einen LIR/RIPE-Account
- mindestens einen Upstream-Provider, der BGP akzeptiert

In Europa läuft das über RIPE NCC.

------------------------------------------------------------------------

## Aufwand eines eigenen AS

Man braucht:

### Organisatorisch

- RIPE-Mitgliedschaft
- Ansprechpartner
- Dokumentation
- Routing-Policy

### Technisch

- Router mit BGP
- Monitoring
- redundante Upstreams

Beispiel:

                 Internet
                    |
           +--------+--------+
           |                 |
       Telekom          anderer ISP
           |                 |
           +-------BGP-------+
                   |
            Genossenschaft AS
                   |
            Wohnblöcke

------------------------------------------------------------------------

## Wann lohnt sich das?

Für:

- 24 Wohnungen → nein
- 200 Wohnungen → vielleicht
- 1000+ Wohnungen → durchaus

Der größte Vorteil ist nicht Bandbreite, sondern **Unabhängigkeit**:

Heute:

    Telekom
     |
    2001:db8:telekom::/56

Wenn ihr kündigt:

    Adresse weg

Mit eigenem Präfix:

    Genossenschaft
     |
    2001:db8:genossenschaft::/48

Providerwechsel:

    Telekom raus
    anderer ISP rein

    Adressen bleiben gleich

------------------------------------------------------------------------

## Eine interessante Zwischenlösung für euch

Ich würde wahrscheinlich so planen:

### Phase 1 (jetzt)

Jeder Block:

- Telekom Business Glasfaser
- eigener Router
- eigenes VLAN-Konzept
- feste IPv6-Präfix-Zuordnung
- zentrale Dokumentation

### Phase 2 (bei Wachstum)

Genossenschaft bekommt:

- eigenes /48 IPv6-Präfix
- eigenes ASN

Dann werden die Blockrouter zu BGP-Standorten.

------------------------------------------------------------------------

## Noch ein praktischer Gedanke

Bei mehreren hundert Wohnungen würde ich nicht nur Internet betrachten. Ein eigenes Gebäude-Netz könnte später auch tragen:

- Hausautomation
- Heizungs-/Energie-Monitoring
- Photovoltaik
- Wallbox-Management
- digitale Klingelanlagen
- Aufzugsüberwachung
- gemeinsames Verwaltungsnetz

Da lohnt sich eine saubere IP- und VLAN-Architektur sehr.

Eine Frage wäre noch entscheidend: **Sind die Wohnblöcke geografisch getrennt (mehrere Straßen/Standorte), oder stehen die alle auf einem zusammenhängenden Gelände?** Das beeinflusst stark, ob man später ein echtes Genossenschafts-Backbone zwischen den Gebäuden aufbauen würde.

## Prompt:

7/20/2026, 4:56:07 PM

> stehen die alle auf einem zusammenhängenden Gelände

ja, die wohnblocks stehen alle direkt nebeneinander in der gleichen straße. was hilft das? sollen wir netzwerkkabel zwischen den wohnblocks legen, und dann brauchen wir noch weniger internetanschlüsse (aber dafür schnellere internetanschlüsse)?

und was genau ist der vorteil von einem eigenen ASN? weil providerwechsel ist unrealistisch, weil die ganzen glasfaserleitungen gehören der telekom, und ein anderer provider müsste eigene leitungen legen...

## Response:

7/20/2026, 4:56:08 PM

Dass die Wohnblöcke direkt nebeneinander stehen, ist tatsächlich ein großer Unterschied – aber nicht unbedingt aus dem Grund, dass ihr "weniger Internetanschlüsse" braucht. Es eröffnet euch vor allem die Möglichkeit, **ein eigenes Gebäudenetz (Campusnetz) aufzubauen**.

Die richtige Frage ist weniger:

> "Wie viele Internetanschlüsse brauchen wir?"

sondern:

> "Wo soll die Grenze zwischen Telekom-Netz und unserem eigenen Netz liegen?"

------------------------------------------------------------------------

## Variante A: Jeder Block eigener Telekom-Anschluss

Das ist die einfachste Lösung:

``` text
Block A
  Telekom 1 Gbit/s
       |
   24 Wohnungen

Block B
  Telekom 1 Gbit/s
       |
   24 Wohnungen

Block C
  Telekom 1 Gbit/s
       |
   24 Wohnungen
```

Vorteile:

- einfach
- jeder Block funktioniert unabhängig
- kein eigenes Backbone
- keine zentrale Netztechnik

Nachteile:

- mehrere Verträge
- mehrere Router
- mehrere Internet-Uplinks
- keine gemeinsame Infrastruktur

------------------------------------------------------------------------

## Variante B: Ein zentraler Internet-Uplink + Hausnetz

Wenn die Gebäude nah beieinander stehen, könnt ihr ein eigenes Backbone bauen:

``` text
                 Telekom
                    |
             Zentrale Technik
                    |
              10/25 Gbit Backbone
          /          |          \
      Block A     Block B     Block C
        |            |           |
     24 Whg       24 Whg      24 Whg
```

Dann braucht ihr nur:

- einen oder wenige Internetanschlüsse
- zentrale Firewall
- zentrale IPv6-Verwaltung

Die Verbindung zwischen den Häusern wäre aber nicht über normale Netzwerkkabel sinnvoll, sondern eher:

- Glasfaser zwischen Gebäuden
- eigene Leerrohre
- LWL-Patchfelder

Kupfer-Ethernet zwischen Gebäuden würde ich nicht empfehlen (Blitzschutz, Potentialunterschiede, Entfernung).

------------------------------------------------------------------------

## Beispielrechnung

Angenommen:

- 10 Wohnblöcke
- 240 Wohnungen

Heute:

    10 × 1 Gbit/s Telekom

Kosten:

    10 × 100 € = 1000 €/Monat

Alternative:

    1 × 10 Gbit/s Business-Anschluss
    +
    eigenes Gebäudenetz

Kosten vielleicht:

    300–500 €/Monat

plus Infrastruktur.

Ob das wirtschaftlich ist, hängt stark von den tatsächlichen Telekompreisen und Baukosten ab.

------------------------------------------------------------------------

# Was bringt ein eigenes ASN?

Das ASN wird oft missverstanden.

Es bedeutet nicht:

> "Wir haben eine eigene Leitung."

Es bedeutet:

> "Das Internet kennt unser Netz als eigenständiges Netz."

Beispiel ohne eigenes ASN:

    Telekom
      |
      IPv6-Präfix der Telekom
      |
      Genossenschaft

Wenn Telekom morgen den Anschluss kündigt:

    IPv6-Präfix weg

Alle Adressen ändern sich.

------------------------------------------------------------------------

Mit eigenem ASN:

    Genossenschaft AS12345
            |
    2001:db8:genossenschaft::/48
            |
       Telekom

Wenn ihr später den Provider wechselt:

    alt:
    Telekom

    neu:
    Provider X

bleibt:

    2001:db8:genossenschaft::/48

gleich.

------------------------------------------------------------------------

## Dein Einwand ist aber völlig richtig

> "Providerwechsel ist unrealistisch, weil die Glasfaserleitungen der Telekom gehören"

Das ist ein sehr wichtiger Punkt.

Ein eigenes ASN lohnt sich **nicht automatisch**, nur weil man viele Wohnungen hat.

Es lohnt sich eher, wenn:

- ihr mehrere Internetanbieter haben wollt
- ihr Redundanz braucht
- ihr eigene Dienste betreibt
- ihr langfristig unabhängig sein wollt

Zum Beispiel:

``` text
Telekom Glasfaser
        |
        |
    Genossenschaft-AS
        |
        |
zweiter Anbieter über andere Trasse
```

Dann könnt ihr bei einer Störung umschalten.

------------------------------------------------------------------------

## Für eure Situation würde ich eher so denken

Bei einer Genossenschaft mit mehreren hundert Wohnungen direkt nebeneinander:

### Sehr sinnvoll:

✅ eigene Glasfaser zwischen den Gebäuden  
✅ zentrale Netzarchitektur  
✅ zentrale IPv6-Adressplanung  
✅ VLAN pro Wohnung  
✅ zentrale Firewall  
✅ Dokumentation

### Vielleicht später:

◻ eigenes ASN  
◻ eigenes IPv6-PI-Präfix  
◻ BGP

------------------------------------------------------------------------

Ein interessanter Mittelweg wäre:

Die Genossenschaft baut ein eigenes **Campusnetz**, aber nutzt weiterhin Telekom als Upstream:

``` text
Genossenschaftsnetz
       |
       |
 Telekom Internet
```

Das bringt schon 90 % der Vorteile, ohne die Komplexität eines AS.

------------------------------------------------------------------------

Eine praktische Frage wäre noch: **Sind die Keller/Technikräume der Wohnblöcke untereinander bereits mit Leerrohren oder Stromtrassen verbunden?** Falls ja, wäre ein gemeinsames Glasfaser-Backbone wahrscheinlich viel realistischer als es zunächst klingt.

## Prompt:

7/20/2026, 5:00:56 PM

> Sind die Keller/Technikräume der Wohnblöcke untereinander bereits mit Leerrohren oder Stromtrassen verbunden?

nein, noch nicht

also ich schätze, der einfachste erste schritt wäre:  
ein internetanschluss pro haus  
alle mieter in einem haus teilen sich diesen anschluss  
mieter in einem haus haben direkte konnektivität zueinander

ich schätze, ein großer vorteil von glasfaser-verbindungen zwischen den häusern wäre die direkte konnektivität zwischen nachbarn in verschiedenen wohnblocks. (und natürlich insgesamt niedrigere betriebskosten, aber dafür müssen erstmal die glasfaser-kabel gelegt werden...)

## Response:

7/20/2026, 5:00:57 PM

Ja, ich glaube, deine Einschätzung trifft die sinnvolle Reihenfolge ziemlich genau. Ihr habt hier eigentlich zwei getrennte Projekte:

1.  **Internetversorgung innerhalb eines Hauses** (relativ einfach)
2.  **Campusnetz zwischen Häusern** (größeres Infrastrukturprojekt)

Ich würde das nicht vermischen.

------------------------------------------------------------------------

## Phase 1: Ein Anschluss pro Haus

Das ist ein sehr guter erster Schritt:

``` text
                 Telekom
                    |
             Glasfaseranschluss
                    |
              Router/Firewall
                    |
             Managed Switch
          /      |       \
      Whg 1   Whg 2   ... Whg 24
```

Jede Wohnung:

- eigener Ethernet-Port
- eigenes VLAN
- eigenes IPv6-/64-Präfix

Also:

    Wohnung 1  → 2001:db8:haus1:001::/64
    Wohnung 2  → 2001:db8:haus1:002::/64
    ...
    Wohnung 24 → 2001:db8:haus1:018::/64

Damit habt ihr bereits:

- saubere Trennung
- eindeutige Zuordnung
- keine NAT-Probleme
- einfache Verwaltung

------------------------------------------------------------------------

## Direkte Kommunikation innerhalb eines Hauses

Das geht problemlos.

Beispiel:

    Wohnung 3
    2001:db8:haus1:003::1234

            ↕ Routing

    Wohnung 7
    2001:db8:haus1:007::5678

Der Router erlaubt einfach Verkehr zwischen den VLANs.

Wichtig: Das ist **kein Layer-2-"gemeinsames Netz"**. Die Wohnungen müssen nicht im selben Ethernet-Broadcast-Netz sein. Sie kommunizieren über IPv6-Routing.

Das ist sogar besser:

- keine Broadcast-Probleme
- weniger Sicherheitsrisiko
- klare Kontrolle

------------------------------------------------------------------------

## Phase 2: Glasfaser zwischen den Häusern

Da wird es interessant.

Angenommen:

    Haus A       Haus B       Haus C

    24 Whg       24 Whg       24 Whg

mit Glasfaser:

                 10/25 Gbit Glasfaser
    Haus A ---------------- Haus B
      |
      |
    Haus C

Dann könnt ihr daraus ein echtes Campusnetz machen.

Vorteile:

### 1. Gemeinsame Internetanbindung

Statt:

    Haus A: 100 €/Monat
    Haus B: 100 €/Monat
    Haus C: 100 €/Monat

könnte man irgendwann:

    ein zentraler 10-Gbit-Anschluss

machen.

------------------------------------------------------------------------

### 2. Direkte Kommunikation zwischen allen Wohnungen

Ohne Backbone:

    Wohnung A1
        |
     Internet
        |
     Wohnung B5

Mit Backbone:

    Wohnung A1
        |
     Hausnetz
        |
     Wohnung B5

Das ist schneller, billiger und unabhängig vom Internetanschluss.

------------------------------------------------------------------------

### 3. Gemeinsame Dienste

Dann werden Dinge interessant wie:

- zentrale Backup-Systeme
- Hausverwaltung
- Energiemanagement
- PV-Anlagen
- Ladeinfrastruktur
- digitale Schwarze Bretter
- interne Telefonie

------------------------------------------------------------------------

## Aber: Glasfaser zwischen Häusern ist ein Bauprojekt

Der Netzwerkteil ist leicht.

Schwierig sind:

- Trassen
- Tiefbau
- Genehmigungen
- Brandschutz
- Gebäudedurchführungen
- Dokumentation

Deshalb würde ich nicht warten, bis ihr ein perfektes Gesamtnetz bauen könnt.

------------------------------------------------------------------------

## Meine Empfehlung für eine Genossenschaft

Ich würde jetzt schon so bauen, dass später erweitert werden kann:

### Pro Haus:

- Technikraum mit 19"-Schrank
- Patchpanel
- Managed Switch
- Router
- Stromversorgung/USV
- Reserveplatz

### Verkabelung:

- pro Wohnung mindestens 1× Cat6A (besser 2×)
- zwischen Technikräumen: **Leerrohre vorsehen**, auch wenn noch keine Glasfaser liegt

Das ist der entscheidende Punkt.

Ein leeres Rohr kostet beim Neubau/bei Sanierung fast nichts. Nachträglich eine Trasse zu bauen ist teuer.

------------------------------------------------------------------------

## Zum Thema "ein Anschluss pro Haus" noch ein Gedanke

Bei 24 Wohnungen ist 1 Gbit/s tatsächlich schon viel.

Wenn jede Wohnung im Durchschnitt 5 % nutzt:

    24 × 50 Mbit/s ≈ 1,2 Gbit/s

Aber die Nutzung ist nicht gleichzeitig gleichmäßig:

- abends Streaming
- Updates
- Cloud-Backups

Ein 1-Gbit/s-Anschluss kann für 24 Wohnungen trotzdem sehr gut funktionieren.

Ein 2,5- oder 10-Gbit-Anschluss wird erst interessant, wenn ihr viele Häuser bündelt.

------------------------------------------------------------------------

Ich würde also eure Roadmap ungefähr so sehen:

**Jahr 1**

- pro Haus ein Business-Glasfaseranschluss
- IPv6 sauber pro Wohnung
- eigene Hausverkabelung

**Jahr 2+**

- Glasfaser-Ringe zwischen Häusern
- zentrale Infrastruktur
- eventuell weniger Internet-Uplinks

Das Schöne daran: Die erste Ausbaustufe ist keine Sackgasse. Die VLAN-/IPv6-Architektur kann später praktisch unverändert in ein größeres Genossenschaftsnetz hineinwachsen.

## Prompt:

7/20/2026, 5:21:25 PM

welchen managed switch brache ich?  
woher kommen die großen preisunterschiede, zum beispiel zwischen 150 und 1000 euro pro switch mit 16 ports?

Ubiquiti UniFiSwitch Pro 24 Rackmount Gigabit Managed Switch, 24x RJ-45, 2x SFP+, 400W PoE++ (USW-Pro-24-PoE)  
Ubiquiti UniFiSwitch Pro 24 Rackmount Gigabit Managed Switch, 24x RJ-45, 2x SFP+, 400W PoE++  
USW-Pro-24-PoE  
Klassifikation: Switch • Typ: Standalone • Montage: Rackmount (via Montagewinkel) • LAN: 8x RJ-45 PoE++ (10/100/1000Base-T, 60W), 16x RJ-45 PoE+ (10/100/1000Base-T, 32W), 2x SFP+ (1G/10Gb/s) • PoE: 8x RJ-45 PoE++ (8x 60W), 16x RJ-45 PoE+ (16x …

(zu wenige) 69 ab € 702,30 \[DE\] computeruniverse

Ubiquiti UniFiSwitch 24 Rackmount Gigabit Managed Switch, 24x RJ-45, 2x SFP, 95W PoE+ (USW-24-POE)  
Ubiquiti UniFiSwitch 24 Rackmount Gigabit Managed Switch, 24x RJ-45, 2x SFP, 95W PoE+  
USW-24-POE  
Klassifikation: Switch • Typ: Standalone • Montage: Rackmount (via Montagewinkel) • LAN: 16x RJ-45 PoE+ (10/100/1000Base-T, 32W), 8x RJ-45 (10/100/1000Base-T), 2x SFP (1Gb/s) • PoE: 16x RJ-45 PoE+ (16x 32W), gesamt: 95W • Backplane: 52Gb/s, …

Bewertung: 5.0 von 5 Sternen  
5.0  
1 Bewertung  
67 ab € 384,89 \[DE\] Cyberport.de  
und 1 weiterer Händler

Ubiquiti UniFiSwitch Pro 48 Rackmount Gigabit Managed Switch, 48x RJ-45, 4x SFP+, 600W PoE++ (USW-Pro-48-PoE)  
Ubiquiti UniFiSwitch Pro 48 Rackmount Gigabit Managed Switch, 48x RJ-45, 4x SFP+, 600W PoE++  
USW-Pro-48-PoE  
Klassifikation: Switch • Typ: Standalone • Montage: Rackmount (via Montagewinkel) • LAN: 8x RJ-45 PoE++ (10/100/1000Base-T, 60W), 40x RJ-45 PoE+ (10/100/1000Base-T, 32W), 4x SFP+ (1G/10Gb/s) • PoE: 8x RJ-45 PoE++ (8x 60W), 40x RJ-45 PoE+ (40x …

(zu wenige)  
1 Testbericht  
66 ab € 1049,00 \[DE\] jks-it

TP-Link SG3400 JetStream Rackmount Gigabit Managed Switch, 24x RJ-45, 4x SFP (TL-SG3428)  
TP-Link SG3400 JetStream Rackmount Gigabit Managed Switch, 24x RJ-45, 4x SFP  
TL-SG3428  
Klassifikation: Switch • Typ: Standalone • Montage: Rackmount (via Montagewinkel) • LAN: 24x RJ-45 (10/100/1000Base-T), 4x SFP (100M/1Gb/s) • Backplane: 56Gb/s, 41.66Mp/s, 9kB Jumboframes, 4.1MB Paketpuffer, 8k MAC-Adressen • Konfiguration: 1x …

(zu wenige) 64 ab € 152,99 \[DE\] eravco

## Response:

7/20/2026, 5:21:33 PM

Für euren konkreten Fall (24 Wohnungen, **ein Port pro Wohnung**, kein WLAN, kein PoE) braucht ihr erstaunlich wenig Switch. Viele der teuren Geräte, die du auflistest, sind für ganz andere Szenarien gebaut.

Die Kernanforderungen sind:

✅ 24× Gigabit-Ethernet-Ports  
✅ VLANs (802.1Q)  
✅ Management (Web/SSH/SNMP)  
✅ ausreichende Backplane-Leistung  
✅ idealerweise 1-4× SFP/SFP+ für spätere Glasfaser-Uplinks  
❌ kein PoE nötig  
❌ kein WLAN  
❌ kein Layer-3-Routing nötig

------------------------------------------------------------------------

## Warum kostet ein Switch 150 € oder 1000 €?

Der Preisunterschied kommt **nicht** hauptsächlich von "mehr VLANs". Fast alle können VLANs.

Die großen Preisfaktoren sind:

------------------------------------------------------------------------

## 1. PoE (sehr großer Faktor)

Beispiel:

Ubiquiti UniFiSwitch Pro 24 PoE:

- 24 Ports
- PoE-Budget bis 400 W
- versorgt:
  - Access Points
  - Kameras
  - VoIP-Telefone
  - Türsprechanlagen

Ein PoE-Netzteil für 400 W ist teuer.

Für euch:

Wohnung → Ethernetdose

braucht **kein PoE**.

------------------------------------------------------------------------

## 2. Layer-3-Funktionen

Billiger Managed Switch:

    MAC-Adresse → Port
    VLAN → Port

Teurer Switch:

    IPv6 Routing
    BGP
    OSPF
    ACLs
    QoS
    Firewall-Funktionen

Das braucht ihr nicht im Switch.

Euer Router macht:

    IPv6-Routing
    Firewall
    Prefix Delegation

------------------------------------------------------------------------

## 3. Redundanz und Enterprise-Funktionen

Teure Geräte bieten:

- zwei Netzteile
- Hot-Swap
- Stackable Switching
- längere Garantie
- Hersteller-Support
- bessere Diagnose
- höhere Betriebstemperatur
- leisere Lüfter
- umfangreiches Monitoring

Für ein Rechenzentrum wichtig.

Für einen Keller im Wohnblock meistens nicht.

------------------------------------------------------------------------

## 4. SFP/SFP+

Das ist für euch interessant.

Beispiel:

    Wohnblock A
          |
          | Glasfaser
          |
    Zentrale Technik

Dafür braucht ihr SFP/SFP+-Ports.

Darum würde ich nicht den billigsten 24-Port-Switch nehmen, sondern einen mit mindestens 2-4 SFP(+).

------------------------------------------------------------------------

# Konkrete Empfehlung für euren ersten Wohnblock

Ich würde ungefähr diese Klasse nehmen:

### Variante günstig und völlig ausreichend

TP-Link JetStream TL-SG3428

- 24× Gigabit
- 4× SFP
- VLAN
- LACP
- STP/RSTP
- Webmanagement
- ~150 €

Der reicht technisch komplett für:

- 24 Wohnungen
- IPv6-Präfixe
- VLAN-Trennung
- 1-Gbit-Internetanschluss

------------------------------------------------------------------------

### Variante "professioneller Hausbetrieb"

MikroTik CRS326-24G-2S+

- 24× Gigabit
- 2× SFP+
- 10-Gbit-Uplink möglich
- VLAN sehr flexibel
- ~200-250 €

Das wäre mein persönlicher Favorit für eine Genossenschaft.

Warum?

Weil ihr später vielleicht genau das wollt:

    Wohnblock
       |
    10 Gbit Glasfaser
       |
    Campusnetz

------------------------------------------------------------------------

### Variante Ubiquiti

Die UniFi-Geräte sind nicht schlecht, aber:

Der USW-Pro-24-PoE für ~700 € ist für euch überdimensioniert.

Ihr bezahlt dort hauptsächlich für:

- PoE
- UniFi-Ökosystem
- Komfortverwaltung
- Enterprise-Features

Nicht für eine bessere IPv6-VLAN-Funktion.

------------------------------------------------------------------------

# Ein wichtiger Punkt: Portanzahl

Du sagst:

> 24 Wohnungen

Ich würde **nicht** exakt 24 Ports kaufen.

Denn du brauchst:

    24 Wohnungen
    + Router-Uplink
    + Management
    + Reserve

Also:

- mindestens 28 Ports
- besser 48 Ports

Ein 48-Port-Switch klingt teuer, aber:

Ein zusätzlicher Switch später ist ärgerlich.

Beispiel:

    Port 1-24  Wohnungen
    Port 25    Router
    Port 26    Management
    Port 27-48 Reserve

Bei Neubau/Erstausbau würde ich eher einen **48-Port-Switch ohne PoE** nehmen.

------------------------------------------------------------------------

# Mein Vorschlag für euren Wohnblock

Technikraum:

    19" Rack

    Router:
    MikroTik RB5009 / OPNsense

            |
            |
    Switch:
    48 Port Managed
            |
            +-- Wohnung 1
            +-- Wohnung 2
            ...
            +-- Wohnung 24
            +-- Reserve

Budget grob:

| Komponente |     Preis |
|------------|----------:|
| Router     | 200-500 € |
| Switch     | 250-500 € |
| Patchpanel |  50-100 € |
| Rack       | 200-500 € |
| USV        | 200-500 € |

Also grob **1000-2000 € pro Haus** für die aktive Technik.

Bei 24 Wohnungen sind das einmalig vielleicht **40-80 € pro Wohnung** - sehr wenig im Vergleich zu den laufenden Internetkosten.

------------------------------------------------------------------------

Noch eine Anmerkung: Für euren späteren Plan mit Glasfaser zwischen Häusern würde ich **SFP+ (10 Gbit/s) statt nur SFP (1 Gbit/s)** bevorzugen. Der Aufpreis ist gering, und damit verbaut ihr euch die Campusnetz-Idee nicht.

## Prompt:

7/20/2026, 6:30:59 PM

ist der managed switch hilfreich gegen "abuse" (missbrauch)?  
zum beispiel, wenn ein mieter den ganzen internetanschluss "verstopft", dann stört er damit alle anderen mieter die den gleichen internetanschluss nutzen.  
kann ich das verhindern mit einem managed switch?

aber andererseits: wenn sonst gerade keiner den internetanschluss nutzt, dann soll auch ein mieter den kompletten anschluss ausnutzen können. also es geht um die frage: was passiert wenn die summe der mieter mehr bandbreite verbrauchen wollen als der anschluss hergibt? wer gewinnt dann?

## Response:

7/20/2026, 6:30:59 PM

Das ist eine sehr gute Frage, und sie zeigt auch den Unterschied zwischen **Switch** und **Router/QoS-System**.

Die kurze Antwort:

**Der Managed Switch allein löst das Problem nicht.**  
Er kann helfen, die Wohnungen sauber zu trennen und einzelne Ports zu kontrollieren, aber die faire Verteilung der Internetbandbreite macht normalerweise der **Router (oder eine Firewall-Appliance)**.

------------------------------------------------------------------------

## Was passiert ohne Begrenzung?

Angenommen:

- Telekom-Anschluss: 1000 Mbit/s
- 24 Wohnungen
- alle hängen dahinter

Ein typischer Aufbau:

                     1 Gbit/s Internet
                           |
                        Router
                           |
                     Managed Switch
              /    /    /       \    \
           Whg1 Whg2 Whg3 ... Whg24

Jetzt:

- Wohnung 1 startet einen großen Download
- Wohnung 2 schaut Netflix
- Wohnung 3 macht Cloud-Backup
- ...

Wenn nichts geregelt ist:

    Wohnung 1:
    900 Mbit/s

    Wohnung 2:
    50 Mbit/s

    Wohnung 3-24:
    fast nichts

Wer gewinnt?

**Der, der am aggressivsten TCP-Verbindungen aufmacht oder am meisten Daten anfordert.**

Das Internet hat keine eingebaute "Wohnungs-Fairness".

------------------------------------------------------------------------

# Was kann der Managed Switch machen?

Er kann zum Beispiel:

### 1. Port-VLAN-Trennung

Sehr wichtig:

    Port 1 → Wohnung 1 VLAN
    Port 2 → Wohnung 2 VLAN
    ...

Damit kann Wohnung 1 nicht einfach:

- DHCP manipulieren
- fremden Traffic sehen
- ARP-Spoofing machen

Das ist ein großer Vorteil.

------------------------------------------------------------------------

### 2. Port-basierte Bandbreitenbegrenzung

Viele Managed Switches können:

    Port 1:
    max 100 Mbit/s

    Port 2:
    max 100 Mbit/s

Aber das ist nicht das, was du willst.

Denn dann passiert:

    Wohnung 1:
    100 Mbit/s

    Internet frei:
    900 Mbit/s ungenutzt

Du verschenkst Kapazität.

------------------------------------------------------------------------

# Die richtige Lösung: QoS / Traffic Shaping im Router

Der Router sieht den gesamten Verkehr:

    Wohnungen
       |
    Switch
       |
    Router
       |
    Internet

Dort kann man Regeln definieren.

Zum Beispiel:

## Variante A: Mindestgerechtigkeit

Jede Wohnung bekommt einen Anteil:

    1000 Mbit/s / 24

    ≈ 40 Mbit/s garantiert

Wenn alle aktiv sind:

    Whg 1: 42
    Whg 2: 39
    Whg 3: 41
    ...

Wenn nur eine aktiv ist:

    Whg 1: 1000

Das entspricht deinem Wunsch:

> Wenn niemand sonst die Leitung nutzt, soll einer alles bekommen.

Das nennt man oft:

- Fair Queuing
- Weighted Fair Queuing
- FQ-CoDel
- CAKE

------------------------------------------------------------------------

## Variante B: harte Obergrenze

Zum Beispiel:

    jede Wohnung maximal 200 Mbit/s

Vorteil:

- eine Wohnung kann nie alles blockieren

Nachteil:

- selbst wenn die Leitung frei ist, bekommt niemand mehr

Für euren Fall würde ich das eher nicht machen.

------------------------------------------------------------------------

## Variante C: Priorisierung

Man könnte sagen:

    normales Surfen:
    hoch

    Bulk-Downloads:
    niedrig

    Backups:
    niedrig

Das ist aber aufwendig und meistens nicht nötig.

------------------------------------------------------------------------

# IPv6 macht das sogar schöner

Da ihr natives IPv6 habt:

Wohnung:

    2001:db8:haus1:001::/64

Router kann Regeln machen:

    Prefix:
    2001:db8:haus1:001::/64

    Queue:
    Wohnung 1

Also keine NAT-Tricks nötig.

------------------------------------------------------------------------

# Welche Hardware macht das?

Hier wird der Router wichtiger als der Switch.

Beispiele:

- OPNsense/pfSense auf Mini-PC
- MikroTik Router
- Ubiquiti UniFi Gateway (je nach Modell)
- professionelle Firewall-Appliances

Ein einfacher Telekom-Router kann das oft nicht ausreichend.

------------------------------------------------------------------------

# Meine Architektur für euren Fall wäre:

                     Telekom
                        |
                  IPv6 Prefix
                        |
                 Router/Firewall
                        |
           Fair Queuing / QoS pro Wohnung
                        |
              Managed Switch
                        |
        +---------------+---------------+
        |               |               |
     Wohnung 1     Wohnung 2       Wohnung 24
     VLAN 101       VLAN 102        VLAN 124

Der Switch sorgt für:

✅ Trennung  
✅ Sicherheit  
✅ saubere Verkabelung

Der Router sorgt für:

✅ Bandbreitenfairness  
✅ IPv6-Präfixe  
✅ Firewall  
✅ Logging/Zuweisung

------------------------------------------------------------------------

Ein interessanter Nebeneffekt: Für eure rechtliche Zielsetzung (Zuordnung einer öffentlichen IPv6-Adresse zu einer Wohnung) ist diese Architektur ebenfalls ideal. Der Router ist die Stelle, an der dokumentiert werden kann:

    Zeitpunkt       IPv6-Präfix              Switch-Port
    10:32:15        2001:db8:haus1:007::/64   Port 7

Damit ist die technische Zuordnung sehr klar.

------------------------------------------------------------------------

Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)
