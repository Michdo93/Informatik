# 💻 Informatik – Kompendium für Hiwis, Praktikanten und Studierende

Dieses Repository sammelt praxisnahes IT-Wissen für alle, die im Labor (Smart Home, Robotik, autonome Systeme) an der **Hochschule Furtwangen (HFU)** mitarbeiten: studentische Hilfskräfte, Praktikantinnen und Praktikanten sowie Studierende in Projekt- und Abschlussarbeiten.

Wer sich für eine Mitarbeit im Labor interessiert, findet Voraussetzungen, Aufgaben und Rahmenbedingungen im Repository **[Praktika im Smart Home Labor](https://github.com/Michdo93/Praktika-Smart-Home-Labor)**. Offene Vorhaben im Labor stehen im **[Smart-Home-Labor-Backlog](https://github.com/Michdo93/Smart-Home-Labor-Backlog)**, Themen für Abschlussarbeiten in **[SmartHome-Ideen](https://github.com/Michdo93/SmartHome-Ideen)**.

Ziel ist eine **schnelle Einarbeitung**: Konventionen, Best Practices und Hintergrundwissen, die man im Studium oft nur am Rande oder gar nicht lernt – die im Laboralltag aber jeden Tag gebraucht werden.

<!-- TOC -->
## Inhaltsverzeichnis

- [Inhalt](#inhalt)
- [Empfohlene Lesereihenfolge](#empfohlene-lesereihenfolge)
- [Konventionen in diesem Repository](#konventionen-in-diesem-repository)
- [Mitmachen](#mitmachen)
<!-- /TOC -->

## Inhalt

| Ordner | Worum geht es? |
| --- | --- |
| [Code-Formatierung](Code-Formatierung/README.md) | Einrückung, Namenskonventionen, Kommentare, Logging, Projektstruktur, Heredoc; Styleguides für **Python, JavaScript, C, C++, C#, Java** |
| [Best Practices](Best%20Practices/README.md) | Dokumentation, **DHCP** (feste IPs), **Zertifikate** (HTTPS von Anfang an), **SSH**, **MQTT**, **Ansible**, **Web-Server**, **Exec Binding** (sicher), **Monitoring** (Nagios/Grafana) |
| [Zugriffskontrolle](Zugriffskontrolle/README.md) | Authentifizierung vs. Autorisierung, Whitelist/Blacklist, ACLs in Betriebssystem, Datenbank, MQTT, Firewall, openHAB |
| [Backup-Strategien](Backup-Strategien/README.md) | Voll/inkrementell/differentiell, Snapshots, VMs vs. Container vs. Datenbanken, Git als Backup, Skripte mit Cron, tar + NAS, Retention, Export/Import, Ansible, Wiederherstellung |
| [Design Pattern](Design%20Pattern/README.md) | Was Entwurfsmuster sind (und was nicht) und alle **23 GoF-Muster** mit Diagramm und Python-Beispiel |
| [Software-Konzepte](Software-Konzepte/README.md) | Templates & Templating, Skeleton/Boilerplate/Scaffolding, Bootstrapping, **Refactoring & Migration**, Orchestrierung & Choreografie |
| [Workarounds & Hacks](Workarounds%20%26%20Hacks/README.md) | Polyfills, Feature Detection, Monkey Patching, Cross-Plattform-Tricks, **Reverse Engineering**, technische Schulden |
| [Linux & Werkzeuge](Linux%20%26%20Werkzeuge/README.md) | **systemd-Services**, Bash-Skripte, Benutzer, Gruppen & Rechte, Cron und systemd-Timer |
| [Netzwerk](Netzwerk/README.md) | IP, Subnetze, Ports, `ping`, ARP, `nmap`, Switches und Schleifen, **HTTP & REST**, Smart-Home-Funkstandards (Zigbee, Z-Wave, Thread, Matter) |
| [Virtualisierung](Virtualisierung/README.md) | **Proxmox** (VM vs. LXC, Backups, Umzug, VM → LXC) und **Docker & Compose** im Betrieb |
| [openHAB](openHAB/README.md) | Laborkonventionen für Betrieb und Konfiguration: Things/Items als Textdateien, Rule Engines, openHAB-Entwurfsmuster, Oberflächen, Sicherheit |
| [Wissenschaftliches Arbeiten](Wissenschaftliches%20Arbeiten/README.md) | Recherche, Quellen bewerten, Zitieren (IEEE/BibTeX), Literaturverwaltung, KI-Werkzeuge, Aufbau einer Abschlussarbeit |
| [Datenbanken](Datenbanken/README.md) | Relationale Modellierung (1:n, n:m, Normalisierung), Zeitreihen und **openHAB Persistence** |
| [KI & Sprachverarbeitung](KI%20%26%20Sprachverarbeitung/README.md) | Machine-Learning-Grundlagen, Intents & Entities, **Fuzzy Matching**, Sprachassistenten (Wakeword, STT, TTS) |
| [Debugging](Debugging/README.md) | Systematisches Vorgehen, Werkzeuge, **Remote-Debugging** auf Pi, VM und Container |
| [Compiler & Build](Compiler%20%26%20Build/README.md) | Compiler, Interpreter, JIT, Linker, Build-Systeme, **Cross-Compiling** |
| [Begriffe & Herkunft](Begriffe%20%26%20Herkunft/README.md) | Warum heißt ein Bug „Bug“? Namensherkunft und Analogien |

---

## Empfohlene Lesereihenfolge

**Erste Woche – bevor die erste Zeile Code entsteht:**

1. [Best Practice Dokumentation](Best%20Practices/Dokumentation.md) – wie wir dokumentieren (Markdown, Englisch, vollständig)
2. [Code-Formatierung](Code-Formatierung/README.md) – und der Styleguide der eigenen Sprache
3. [Linux & Werkzeuge](Linux%20%26%20Werkzeuge/README.md) und [Netzwerk-Grundlagen](Netzwerk/Netzwerk-Grundlagen.md) – das Handwerkszeug für jedes Laborsystem
4. [DHCP](Best%20Practices/DHCP.md), [SSH](Best%20Practices/SSH.md), [Zertifikate](Best%20Practices/Zertifikate.md) – bevor ein neues Gerät ins Netz kommt
5. [Technische Schulden](Workarounds%20%26%20Hacks/Technische%20Schulden.md) – warum Provisorien teuer werden

**Sobald Geräte und Dienste betrieben werden:**

6. [MQTT](Best%20Practices/MQTT.md) und [Zugriffskontrolle](Zugriffskontrolle/README.md)
7. [Ansible](Best%20Practices/Ansible.md), [Cron & systemd-Timer](Linux%20%26%20Werkzeuge/Cron%20%26%20systemd-Timer.md), [HTTP & REST](Netzwerk/HTTP%20%26%20REST.md) und [Web-Server & Deployment](Best%20Practices/Web-Server%20%26%20Deployment.md)
8. [Backup-Strategien](Backup-Strategien/README.md) – **bevor** etwas verloren geht – und [Virtualisierung](Virtualisierung/README.md) für die Arbeit mit Proxmox und Docker

**Zum Vertiefen und Nachschlagen:**

9. [Debugging](Debugging/README.md), [Compiler & Build](Compiler%20%26%20Build/README.md)
10. [Design Pattern](Design%20Pattern/README.md), [Software-Konzepte](Software-Konzepte/README.md), [Datenbanken](Datenbanken/README.md), [KI & Sprachverarbeitung](KI%20%26%20Sprachverarbeitung/README.md) – besonders vor Projekt- und Abschlussarbeiten
11. [openHAB](openHAB/README.md) – Laborkonventionen für Betrieb und Konfiguration
12. [Wissenschaftliches Arbeiten](Wissenschaftliches%20Arbeiten/README.md) – für Projekt- und Abschlussarbeiten
11. [Workarounds & Hacks](Workarounds%20%26%20Hacks/README.md), [Begriffe & Herkunft](Begriffe%20%26%20Herkunft/README.md)

---

## Konventionen in diesem Repository

* Alle Dokumente sind **Markdown** und haben ein **Inhaltsverzeichnis** am Anfang.
* Diagramme sind als **Mermaid** eingebettet und werden von GitHub direkt dargestellt.
* Erklärtexte sind auf **Deutsch**; Code, Kommentare im Code und Projekt-Dokumentation in den eigenen Repositories auf **Englisch** (siehe [Dokumentation](Best%20Practices/Dokumentation.md)).
* Beispielcode ist eine **Vorlage**: IP-Adressen, Hostnamen, Item-Namen, Pfade und Schwellenwerte an die eigene Installation anpassen.

---

## Mitmachen

Fehler gefunden oder etwas fehlt? Gerne ein Issue anlegen oder einen Pull Request stellen. Neue Dokumente bitte im Stil der bestehenden anlegen (Titel, kurze Einleitung, `<!-- TOC -->`-Marker, Abschnitte mit `---` getrennt) und im README des jeweiligen Ordners verlinken.

---
