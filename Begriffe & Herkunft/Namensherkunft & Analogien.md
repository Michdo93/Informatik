# 🏷️ Namensherkunft & Analogien

Viele Begriffe der Informatik wirken seltsam – bis man ihre Herkunft kennt. Dann werden sie zu **Eselsbrücken**. Dieses Dokument sammelt die wichtigsten Begriffe mit ihrer Geschichte und einer Analogie.

> **Hinweis zur Quellenlage:** Bei manchen Begriffen gibt es konkurrierende oder unbelegte Erklärungen. Das ist jeweils mit „vermutlich“, „angeblich“ oder „umstritten“ gekennzeichnet – solche Geschichten bitte nicht als gesicherte Fakten in Abschlussarbeiten zitieren.

<!-- TOC -->
## Inhaltsverzeichnis

- [Fehler und Fehlersuche](#fehler-und-fehlersuche)
  - [Bug](#bug)
  - [Debugging](#debugging)
  - [Heisenbug, Bohrbug, Mandelbug, Schrödingbug](#heisenbug-bohrbug-mandelbug-schrödingbug)
  - [Rubber Duck Debugging](#rubber-duck-debugging)
  - [Patch](#patch)
- [Programme und Prozesse](#programme-und-prozesse)
  - [Daemon](#daemon)
  - [Kernel und Shell](#kernel-und-shell)
  - [Booten](#booten)
  - [Thread](#thread)
  - [Fork](#fork)
  - [Zombie- und Waisenprozesse](#zombie--und-waisenprozesse)
- [Netzwerk und Sicherheit](#netzwerk-und-sicherheit)
  - [Ping](#ping)
  - [Firewall](#firewall)
  - [Trojaner (Trojanisches Pferd)](#trojaner-trojanisches-pferd)
  - [Wurm und Virus](#wurm-und-virus)
  - [Spam](#spam)
  - [Cookie](#cookie)
  - [Phishing](#phishing)
  - [Honeypot](#honeypot)
  - [Sandbox](#sandbox)
  - [Whitelist / Blacklist](#whitelist--blacklist)
- [Daten und Speicher](#daten-und-speicher)
  - [Bit und Byte](#bit-und-byte)
  - [Cache](#cache)
  - [Buffer](#buffer)
  - [Pixel](#pixel)
  - [Stack und Heap](#stack-und-heap)
  - [Dump](#dump)
- [Software und Werkzeuge](#software-und-werkzeuge)
  - [Python](#python)
  - [Java](#java)
  - [Git](#git)
  - [Linux](#linux)
  - [Debian](#debian)
  - [Ubuntu](#ubuntu)
  - [Raspberry Pi](#raspberry-pi)
  - [openHAB](#openhab)
  - [MQTT](#mqtt)
  - [Ansible](#ansible)
  - [Docker und Kubernetes](#docker-und-kubernetes)
  - [Wiki](#wiki)
  - [Polyfill](#polyfill)
  - [Cron](#cron)
- [Programmierkultur](#programmierkultur)
  - [Foo, Bar, Baz](#foo-bar-baz)
  - [Hello, World!](#hello-world)
  - [Hack und Hacker](#hack-und-hacker)
  - [Monkey Patch](#monkey-patch)
  - [Easter Egg](#easter-egg)
  - [Yak Shaving](#yak-shaving)
  - [Bikeshedding](#bikeshedding)
  - [Cargo Cult Programming](#cargo-cult-programming)
  - [Spaghetti-Code](#spaghetti-code)
  - [Silver Bullet](#silver-bullet)
  - [Dogfooding](#dogfooding)
- [Eselsbrücken für Konzepte](#eselsbrücken-für-konzepte)
<!-- /TOC -->

## Fehler und Fehlersuche

### Bug

* *Bug* (Käfer, Insekt) war schon im 19. Jahrhundert ein Ausdruck für **technische Macken**. **Thomas Edison** schrieb 1878 in Briefen von „Bugs“ in seinen Erfindungen.
* Berühmt wurde der Begriff durch einen Vorfall am **9. September 1947**: Am **Harvard Mark II** fand das Team um **Grace Hopper** eine **Motte**, die in einem Relais klemmte. Sie wurde ins Logbuch geklebt mit dem Vermerk *„First actual case of bug being found.“* – der Witz war gerade, dass man endlich einen **echten** Käfer gefunden hatte. Das Logbuch liegt heute im Smithsonian National Museum of American History.
* **Analogie:** Ein Insekt im Getriebe – klein, versteckt, aber es bringt alles durcheinander.

### Debugging

Das Entfernen der Bugs. Siehe [Debugging](../Debugging/README.md).

### Heisenbug, Bohrbug, Mandelbug, Schrödingbug

Wortspiele mit Physikern und Mathematikern: Der **Heisenbug** verschwindet, sobald man ihn beobachtet (Heisenbergs Unschärferelation). Der **Bohrbug** ist solide reproduzierbar (Bohrs Atommodell). Der **Mandelbug** ist chaotisch komplex (Mandelbrot-Menge). Der **Schrödingbug** existiert erst, wenn man merkt, dass der Code nie hätte funktionieren dürfen (Schrödingers Katze). Details: [Vorgehen beim Debugging](../Debugging/Vorgehen%20beim%20Debugging.md).

### Rubber Duck Debugging

Aus dem Buch *The Pragmatic Programmer* (1999): Ein Programmierer erklärt seinen Code Zeile für Zeile einer **Gummiente** – und findet dabei den Fehler selbst.

### Patch

Wörtlich ein **Flicken**. Programme auf **Lochstreifen** wurden korrigiert, indem man Stücke herausschnitt und einen neuen Streifen **aufklebte** bzw. Löcher in Lochkarten überklebte.

---

## Programme und Prozesse

### Daemon

Hintergrundprozesse (sshd, crond, mosquitto) heißen **Daemons**. Der Name stammt aus dem **MIT-Projekt MAC** (1963) und bezieht sich auf **Maxwells Dämon** – ein Gedankenexperiment, in dem ein unsichtbares Wesen unermüdlich im Hintergrund Moleküle sortiert. Gemeint ist im griechischen Sinn (*daimon*) ein **hilfreicher Geist**, kein böser „Dämon“. Das „d“ am Ende vieler Dienstnamen (`sshd`, `httpd`) steht für *daemon*.

### Kernel und Shell

Bild einer **Nuss** oder Frucht: Der **Kernel** (Kern) ist das Innerste des Betriebssystems, das direkt mit der Hardware spricht. Die **Shell** (Schale) umgibt ihn und ist die Schnittstelle zum Menschen.

### Booten

Kurz für *Bootstrapping* – sich „an den eigenen Stiefelschlaufen hochziehen“. Siehe [Bootstrapping](../Software-Konzepte/Bootstrapping.md).

### Thread

Engl. **Faden**. Ein Programm (Prozess) kann mehrere „Ausführungsfäden“ haben, die parallel laufen – wie mehrere Fäden in einem Seil.

### Fork

Engl. **Gabel / Weggabelung**: Ein Prozess teilt sich in zwei (`fork()`), oder ein Software-Projekt spaltet sich ab (GitHub-Fork).

### Zombie- und Waisenprozesse

Ein **Zombie** ist ein beendeter Prozess, dessen Exit-Status der Elternprozess noch nicht abgeholt hat – „tot, aber noch da“. Ein **Waise** (orphan) ist ein Prozess, dessen Eltern beendet wurden; er wird von `init`/systemd „adoptiert“.

---

## Netzwerk und Sicherheit

### Ping

Nach dem Geräusch eines **Sonars** (U-Boot-Ortung): Ein Signal wird ausgesendet und das Echo gemessen. Mike Muuss schrieb das Programm 1983 und wählte den Namen nach diesem Klang. Die oft genannte Auflösung „Packet Internet Groper“ ist ein nachträgliches **Backronym**.

### Firewall

Eine **Brandschutzmauer** in Gebäuden (oder zwischen Motor und Fahrgastraum im Auto) verhindert, dass sich ein Feuer ausbreitet. Die Firewall soll verhindern, dass sich Angriffe von einem Netz ins andere ausbreiten.

### Trojaner (Trojanisches Pferd)

Aus der griechischen Sage: Die Griechen versteckten Soldaten in einem hölzernen Pferd, das die Trojaner als Geschenk in ihre Stadt holten. Ein Trojaner ist ein **scheinbar nützliches Programm** mit verstecktem Schadcode. (Streng genommen müsste es „Trojanisches Pferd“ heißen – die Trojaner waren ja die Opfer.)

### Wurm und Virus

* **Virus:** Wie ein biologisches Virus braucht er einen **Wirt** (ein anderes Programm/eine Datei), um sich zu vermehren. Den Begriff prägten Fred Cohen und sein Betreuer Leonard Adleman 1983/84.
* **Wurm:** Verbreitet sich **selbstständig** über das Netzwerk, ohne Wirt. Der Name geht auf den Science-Fiction-Roman *The Shockwave Rider* (John Brunner, 1975) zurück, in dem ein „Tapeworm“ (Bandwurm) durch ein Netz kriecht.

### Spam

Nach einem Sketch von **Monty Python** (1970): In einem Café besteht jedes Gericht aus *Spam* (einer Dosenfleisch-Marke), und eine Gruppe Wikinger singt immer lauter „Spam, Spam, Spam …“, bis kein Gespräch mehr möglich ist. Genau so übertönen Massenmails die eigentliche Kommunikation.

### Cookie

Abgeleitet von **„Magic Cookie“**, einem älteren Unix-Begriff für ein **Datenpaket, das ein Programm empfängt und unverändert zurückgibt**. Lou Montulli übernahm den Begriff 1994 bei Netscape für Browser-Cookies. Die verbreitete Analogie zu **Glückskeksen** (eine Nachricht darin) oder zur Brotkrumen-Spur passt, ist aber nicht die belegte Herkunft.

### Phishing

Von *fishing* (Angeln) – man wirft einen Köder aus und wartet, wer anbeißt. Das „ph“ ist eine Anspielung auf das **Phreaking** (Manipulation von Telefonnetzen) der Hacker-Szene der 1970er.

### Honeypot

Ein **Honigtopf**, der Angreifer anlockt wie Bären oder Wespen – ein absichtlich verwundbar wirkendes System zur Beobachtung von Angriffen.

### Sandbox

Ein **Sandkasten**, in dem Kinder spielen können, ohne etwas kaputt zu machen. Code läuft isoliert, ohne Zugriff auf das restliche System.

### Whitelist / Blacklist

Siehe [Whitelist & Blacklist](../Zugriffskontrolle/Whitelist%20%26%20Blacklist.md) – inklusive der heute bevorzugten Begriffe **Allowlist / Denylist**.

---

## Daten und Speicher

### Bit und Byte

* **Bit** = *binary digit* (Binärziffer). Den Begriff verwendete **Claude Shannon** 1948 und schrieb ihn John Tukey zu.
* **Byte** prägte **Werner Buchholz** 1956 bei IBM – eine absichtlich falsch geschriebene Form von *bite* (Bissen), damit man es nicht mit *bit* verwechselt. Ein Byte ist also ein „Bissen“ Bits.
* Ein halbes Byte (4 Bit) heißt scherzhaft **Nibble** (Knabberei).

### Cache

Französisch *cacher* = **verstecken**. Ursprünglich ein **Versteck für Vorräte** (Trapper, Expeditionen). Ein Cache ist ein schneller Zwischenspeicher, in dem man Daten „versteckt“, um sie nicht wieder vom langsamen Speicher holen zu müssen.

### Buffer

Engl. **Puffer**, wie der Puffer zwischen Eisenbahnwaggons: Er gleicht Unterschiede (in der Geschwindigkeit von Erzeuger und Verbraucher) aus.

### Pixel

Kurzform von ***pic*ture *el*ement** (pics = pictures).

### Stack und Heap

* **Stack** = **Stapel** (wie ein Tellerstapel in der Mensa): Was zuletzt drauf gelegt wurde, kommt zuerst wieder runter (LIFO).
* **Heap** = **Haufen**: Speicher, aus dem man sich beliebig große Stücke nimmt – ohne Ordnung. (Nicht zu verwechseln mit der Datenstruktur *Heap*.)

### Dump

Engl. **Müllkippe / Abladen**: Speicherinhalt oder Datenbank wird „abgekippt“ (`pg_dump`, Core Dump).

---

## Software und Werkzeuge

### Python

Benannt nach **Monty Python's Flying Circus**, nicht nach der Schlange. Guido van Rossum las beim Entwurf (1989) die Drehbücher der Serie und wollte einen kurzen, etwas mysteriösen Namen. Deshalb heißen Beispielvariablen in der Python-Doku oft `spam` und `eggs`.

### Java

Benannt nach **Kaffee** von der indonesischen Insel Java – angeblich, weil das Team viel davon trank. Vorher hieß die Sprache **Oak** (nach einer Eiche vor James Goslings Büro), der Name war aber markenrechtlich belegt. Daher auch die Kaffeetasse im Logo und Begriffe wie **JavaBeans**.

### Git

Linus Torvalds nannte es scherzhaft nach britischem Slang *git* = **Blödmann/Idiot**: „Ich benenne alle meine Projekte nach mir selbst. Erst Linux, jetzt Git.“ Die README von Git bietet augenzwinkernd weitere Deutungen an („global information tracker“ – wenn es funktioniert).

### Linux

Von **Linus** + Unix. Torvalds wollte es ursprünglich „Freax“ nennen; der Admin des FTP-Servers legte das Verzeichnis aber unter `linux` an.

### Debian

Aus den Vornamen **Debra** Lynn und **Ian** Murdock (Gründer, 1993). Die Debian-Releases heißen nach Figuren aus **Toy Story** (Buster, Bullseye, Bookworm, Trixie …); die instabile Version heißt immer **Sid** – der Junge, der Spielzeug kaputt macht.

### Ubuntu

Ein Wort aus den Sprachen Zulu und Xhosa, sinngemäß **„Menschlichkeit gegenüber anderen“** / „Ich bin, weil wir sind“.

### Raspberry Pi

**Raspberry** (Himbeere) in der Tradition von Obst-Namen für Computer (Apple, Apricot, Tangerine). **Pi** steht für **Python**, das als Hauptprogrammiersprache gedacht war.

### openHAB

**open Home Automation Bus** – ein offener „Bus“ (Datenschiene), über den alle Geräte im Haus Nachrichten austauschen.

### MQTT

Ursprünglich **MQ Telemetry Transport** (MQ von IBMs Produktfamilie *MQSeries*). Entwickelt 1999 von Andy Stanford-Clark (IBM) und Arlen Nipper zur Überwachung von **Ölpipelines** über teure Satellitenverbindungen – daher das extrem schlanke Protokoll. Heute gilt der Name offiziell nicht mehr als Abkürzung.

### Ansible

Aus Ursula K. Le Guins Roman *Rocannon's World* (1966): ein **fiktives Gerät zur sofortigen Kommunikation** über beliebige Entfernungen. Passend für ein Werkzeug, das viele Rechner gleichzeitig anspricht.

### Docker und Kubernetes

* **Docker** = **Hafenarbeiter**, der Container verlädt. Das Logo ist ein Wal, der Container trägt.
* **Kubernetes** = griechisch **Steuermann**, Lotse (κυβερνήτης) – verwandt mit *Kybernetik* und *Gouverneur*. Abgekürzt **K8s** (K + 8 Buchstaben + s). Das Logo ist ein Steuerrad.

### Wiki

Hawaiianisch *wiki wiki* = **schnell**. Ward Cunningham benannte 1995 sein WikiWikiWeb nach dem Shuttlebus am Flughafen Honolulu. Von ihm stammt auch die Metapher der [Technischen Schulden](../Workarounds%20%26%20Hacks/Technische%20Schulden.md).

### Polyfill

Nach der britischen Spachtelmasse **Polyfilla** – Löcher in der Wand (des Browsers) zuspachteln. Siehe [Polyfill & Shim](../Workarounds%20%26%20Hacks/Polyfill%20%26%20Shim.md).

### Cron

Griechisch *chronos* = **Zeit**. Siehe [Cron & systemd-Timer](../Linux%20%26%20Werkzeuge/Cron%20%26%20systemd-Timer.md).

---

## Programmierkultur

### Foo, Bar, Baz

Standard-Platzhalternamen. Vermutlich vom militärischen Slang **FUBAR** (*„Fouled/F… Up Beyond All Recognition“*) aus dem Zweiten Weltkrieg; *foo* ist aber noch älter (Comics der 1930er, „Smokey Stover“). Am MIT (Tech Model Railroad Club) setzte sich *foo* als Platzhalter durch.

### Hello, World!

Durch Brian Kernighan bekannt geworden (Tutorial zur Sprache B 1972/73, dann im Buch *The C Programming Language*, 1978). Seitdem das erste Programm in jeder neuen Sprache.

### Hack und Hacker

Am MIT ein **cleverer, verspielter Trick**. Siehe [Begriffe: Workaround, Hack & Co.](../Workarounds%20%26%20Hacks/Begriffe.md).

### Monkey Patch

Aus *Guerrilla Patch* → *Gorilla* → *Monkey*. Siehe [Monkey Patching](../Workarounds%20%26%20Hacks/Monkey%20Patching.md).

### Easter Egg

Ein verstecktes Feature, wie **Ostereier, die man suchen muss**. Bekannt wurde der Begriff durch das Atari-Spiel *Adventure* (1980): Entwickler Warren Robinett versteckte darin seinen Namen, weil Atari die Entwickler nicht nennen wollte.

### Yak Shaving

Man will eigentlich nur X tun, muss dafür aber Y erledigen, dafür Z, … und am Ende **rasiert man einen Yak** und weiß nicht mehr, warum. Der Begriff stammt aus dem MIT AI Lab (um 2000) und spielt auf eine Folge der Zeichentrickserie *Ren & Stimpy* an. Klassisches Beispiel: Um einen Sensor auszulesen, muss man eine Bibliothek aktualisieren, dafür Python, dafür das Betriebssystem …

### Bikeshedding

Nach **Parkinsons Gesetz der Trivialität** (C. Northcote Parkinson, 1957): Ein Komitee genehmigt ein Atomkraftwerk in Minuten, diskutiert aber stundenlang über die Farbe des **Fahrradschuppens** – weil jeder dazu eine Meinung hat. In der IT: Lange Debatten über Tabs vs. Spaces, während die Architekturfrage untergeht. (Siehe auch [Code-Formatierung](../Code-Formatierung/README.md) – genau deshalb legt man Styleguides **einmal** fest.)

### Cargo Cult Programming

Nach den **Cargo-Kulten** in Melanesien: Nach dem Zweiten Weltkrieg bauten Inselbewohner Landebahnen und Funktürme aus Holz nach, in der Hoffnung, dass wieder Flugzeuge mit Fracht landen. In der IT: Code oder Rituale übernehmen, **ohne zu verstehen, warum** sie funktionieren („Das stand so auf Stack Overflow“).

### Spaghetti-Code

Code, dessen Kontrollfluss so verschlungen ist wie ein **Teller Spaghetti** – typischerweise durch viele `goto`-Sprünge. Verwandt: **Lasagne-Code** (zu viele Schichten), **Ravioli-Code** (zu viele winzige, isolierte Klassen).

### Silver Bullet

Nach Fred Brooks' Aufsatz *No Silver Bullet* (1986): Nur eine **Silberkugel** tötet den Werwolf – aber für Softwareprobleme gibt es keine einzelne Wunderlösung.

### Dogfooding

*„Eating your own dog food“* – die eigene Software selbst benutzen. Angeblich nach einem Werbespot, in dem ein Hundefutter-Hersteller sein Produkt seinem eigenen Hund gab.

---

## Eselsbrücken für Konzepte

| Konzept | Analogie |
| --- | --- |
| **Client / Server** | Gast und Kellner im Restaurant |
| **IP-Adresse / Port** | Hausadresse / Wohnungsnummer im Haus |
| **DNS** | Telefonbuch: Name → Nummer |
| **DHCP** | Hotelrezeption, die Zimmer (IPs) vergibt; Reservierung = Stammgast mit festem Zimmer (→ [DHCP](../Best%20Practices/DHCP.md)) |
| **Router** | Postverteilzentrum |
| **Switch** | Hausverteiler / Briefkastenanlage im Haus |
| **Verschlüsselung (TLS)** | Versiegelter, blickdichter Umschlag |
| **Zertifikat** | Personalausweis, ausgestellt von einer vertrauenswürdigen Behörde (CA) (→ [Zertifikate](../Best%20Practices/Zertifikate.md)) |
| **Public / Private Key** | Offenes Vorhängeschloss (jeder kann zuschließen) / Schlüssel (nur ich kann aufschließen) |
| **Hash** | Fingerabdruck einer Datei |
| **Authentifizierung / Autorisierung** | Ausweis zeigen / auf der Gästeliste stehen (→ [AuthN & AuthZ](../Zugriffskontrolle/Authentifizierung%20%26%20Autorisierung.md)) |
| **MQTT Publish/Subscribe** | Zeitschriften-Abo über einen Kiosk (Broker) |
| **API** | Speisekarte: Was man bestellen kann, ohne die Küche zu kennen |
| **Git-Commit** | Speicherstand im Videospiel |
| **Git-Branch** | Paralleluniversum, das man später wieder zusammenführen kann |
| **Container vs. VM** | Wohnung in einem Mehrfamilienhaus (teilt Fundament/Kernel) vs. eigenes Einfamilienhaus |
| **Compiler / Interpreter** | Buchübersetzer / Simultandolmetscher (→ [Compiler & Build](../Compiler%20%26%20Build/README.md)) |
| **Linker** | Setzer, der die Seitenverweise eines Buches einträgt |
| **Backup: Voll / Inkrementell** | Ganzes Fotoalbum kopieren / nur neue Fotos seit dem letzten Mal (→ [Backup-Arten](../Backup-Strategien/Backup-Arten.md)) |

---
