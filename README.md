# 💻 Informatik – Kompendium für Hiwis, Praktikanten und Studierende

Dieses Repository sammelt praxisnahes IT-Wissen für alle, die im Labor (Smart Home, Robotik, autonome Systeme) an der **Hochschule Furtwangen (HFU)** mitarbeiten: studentische Hilfskräfte, Praktikantinnen und Praktikanten sowie Studierende in Projekt- und Abschlussarbeiten.

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
| [Best Practices](Best%20Practices/README.md) | Dokumentation, **DHCP** (feste IPs), **Zertifikate** (HTTPS von Anfang an), **SSH** (Port, Keys, sshpass), **MQTT** (TLS, Passwörter, ACL), **Ansible** (Rollen, Gruppen, Inventar) |
| [Zugriffskontrolle](Zugriffskontrolle/README.md) | Authentifizierung vs. Autorisierung, Whitelist/Blacklist, ACLs in Betriebssystem, Datenbank, MQTT, Firewall, openHAB |
| [Backup-Strategien](Backup-Strategien/README.md) | Voll/inkrementell/differentiell, Snapshots, VMs vs. Container vs. Datenbanken, Git als Backup, Skripte mit Cron, tar + NAS, Retention, Export/Import, Ansible, Wiederherstellung |
| [Design Pattern](Design%20Pattern/README.md) | Was Entwurfsmuster sind (und was nicht) und alle **23 GoF-Muster** mit Diagramm und Python-Beispiel |
| [Software-Konzepte](Software-Konzepte/README.md) | Templates & Templating, Skeleton/Boilerplate/Scaffolding, Bootstrapping, Orchestrierung & Choreografie |
| [Workarounds & Hacks](Workarounds%20%26%20Hacks/README.md) | Polyfills, Feature Detection, Monkey Patching, Cross-Plattform-Tricks, technische Schulden |
| [Linux & Werkzeuge](Linux%20%26%20Werkzeuge/README.md) | Cron und systemd-Timer |
| [Debugging](Debugging/README.md) | Systematisches Vorgehen, Werkzeuge, **Remote-Debugging** auf Pi, VM und Container |
| [Compiler & Build](Compiler%20%26%20Build/README.md) | Compiler, Interpreter, JIT, Linker, Build-Systeme, **Cross-Compiling** |
| [Begriffe & Herkunft](Begriffe%20%26%20Herkunft/README.md) | Warum heißt ein Bug „Bug“? Namensherkunft und Analogien |

---

## Empfohlene Lesereihenfolge

**Erste Woche – bevor die erste Zeile Code entsteht:**

1. [Best Practice Dokumentation](Best%20Practices/Dokumentation.md) – wie wir dokumentieren (Markdown, Englisch, vollständig)
2. [Code-Formatierung](Code-Formatierung/README.md) – und der Styleguide der eigenen Sprache
3. [DHCP](Best%20Practices/DHCP.md), [SSH](Best%20Practices/SSH.md), [Zertifikate](Best%20Practices/Zertifikate.md) – bevor ein neues Gerät ins Netz kommt
4. [Technische Schulden](Workarounds%20%26%20Hacks/Technische%20Schulden.md) – warum Provisorien teuer werden

**Sobald Geräte und Dienste betrieben werden:**

5. [MQTT](Best%20Practices/MQTT.md) und [Zugriffskontrolle](Zugriffskontrolle/README.md)
6. [Ansible](Best%20Practices/Ansible.md) und [Cron & systemd-Timer](Linux%20%26%20Werkzeuge/Cron%20%26%20systemd-Timer.md)
7. [Backup-Strategien](Backup-Strategien/README.md) – **bevor** etwas verloren geht

**Zum Vertiefen und Nachschlagen:**

8. [Debugging](Debugging/README.md), [Compiler & Build](Compiler%20%26%20Build/README.md)
9. [Design Pattern](Design%20Pattern/README.md), [Software-Konzepte](Software-Konzepte/README.md)
10. [Workarounds & Hacks](Workarounds%20%26%20Hacks/README.md), [Begriffe & Herkunft](Begriffe%20%26%20Herkunft/README.md)

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
