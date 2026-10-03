# 🏠 openHAB: Betrieb & Konfiguration

**openHAB** (open Home Automation Bus) ist das Herzstück des Smart Home Labors. Dieses Kapitel fasst zusammen, wie wir openHAB **betreiben und konfigurieren** – unabhängig von einzelnen Geräten. Es ersetzt nicht die offizielle Dokumentation, sondern hält die Laborkonventionen fest.

<!-- TOC -->
## Inhaltsverzeichnis

- [Grundbegriffe](#grundbegriffe)
- [Textkonfiguration statt UI](#textkonfiguration-statt-ui)
  - [Namenskonventionen](#namenskonventionen)
- [Regeln (Rules)](#regeln-rules)
  - [Verbreitete openHAB-Entwurfsmuster](#verbreitete-openhab-entwurfsmuster)
- [Oberflächen](#oberflächen)
- [Betrieb](#betrieb)
- [Sicherheit](#sicherheit)
- [Checkliste für ein neues Gerät](#checkliste-für-ein-neues-gerät)
<!-- /TOC -->

## Grundbegriffe

| Begriff | Bedeutung |
| --- | --- |
| **Binding** | Erweiterung (Add-on), die openHAB mit einer Geräteklasse oder einem Protokoll verbindet |
| **Thing** | Ein konkretes Gerät oder ein Dienst |
| **Channel** | Eine einzelne Funktion eines Things (z. B. Helligkeit) |
| **Item** | Die steuerbare/abfragbare Größe, mit der openHAB intern arbeitet |
| **Link** | Verbindung zwischen Channel und Item |
| **Rule** | Automatisierungsregel |
| **Persistence** | Speicherung von Zustandsverläufen (→ [Zeitreihen & openHAB Persistence](../Datenbanken/Zeitreihen%20%26%20openHAB%20Persistence.md)) |
| **Sitemap / MainUI** | Bedienoberflächen |
| **Semantisches Modell** | Einordnung der Items nach Ort, Gerät und Eigenschaft (Location/Equipment/Point/Property) |

**Command vs. State:** Ein **Command** fordert eine Aktion an (Gerät schalten), ein **State** meldet den Zustand. Dieselbe Unterscheidung gilt in der REST API (POST = Command, PUT `/state` = State-Update, → [HTTP & REST](../Netzwerk/HTTP%20%26%20REST.md)).

---

## Textkonfiguration statt UI

Im Labor konfigurieren wir **textbasiert** (Dateien unter `$OPENHAB_CONF`), nicht über die UI. Der Grund: Textdateien landen im **Git-Backup**, sind versioniert, vergleichbar und reproduzierbar.

| Ort | Inhalt |
| --- | --- |
| `things/*.things` | Things |
| `items/*.items` | Items |
| `rules/*.rules`, `automation/…` | Regeln (DSL bzw. Scripting) |
| `sitemaps/*.sitemap` | Sitemaps |
| `persistence/*.persist` | Persistenz-Strategien |
| `transform/*` | Transformationen (MAP, JSONPATH, JS …) |
| `misc/exec.whitelist` | Erlaubte Exec-Befehle (→ [Exec Binding & Remote-Ausführung](../Best%20Practices/Exec-Binding%20%26%20Remote-Ausführung.md)) |

Zusätzlich gibt es eine interne Datenbank (`$OPENHAB_USERDATA/jsondb/`), in der UI-erstellte Objekte liegen. Deshalb sichern wir auch `userdata` (bzw. die relevanten JSON-Dateien) als zweite, textbasierte Ebene.

> **Konsistenz:** Dinge **entweder** per Textdatei **oder** per UI anlegen, nicht gemischt – sonst entstehen doppelte Items und schwer auffindbare Inkonsistenzen.

### Namenskonventionen

Sprechende, systematische Namen erleichtern Regeln, Fuzzy Matching und Sprachsteuerung. Im Labor hat sich ein Präfix-Schema bewährt, z. B. `i` für Item, `g` für Gruppe, `t` für Thing, gefolgt von Raum, Gerät und Funktion:

```text
tKueche_Sonos_Lautsprecher        (Thing)
gKueche_Licht                     (Gruppe)
iKueche_Hue_Lampe1_Schalter       (Item: Switch)
iKueche_Hue_Lampe1_Dimmer         (Item: Dimmer)
```

---

## Regeln (Rules)

openHAB bietet mehrere **Rule Engines**:

| Engine | Sprache | Hinweise |
| --- | --- | --- |
| **Rules DSL** | eigene DSL (Xtend) | Einfach, weit verbreitet, viele Beispiele; bleibt unterstützt |
| **JavaScript Scripting** | JS (GraalJS) | Moderne JS-Features |
| **Python 3 Scripting** | Python 3 (GraalPy) | Löst das alte Jython ab |
| **jRuby, Groovy, Blockly** | – | weitere Optionen |

Im Labor migrieren wir auf **Python 3 Scripting** (mit Rules DSL als Rückfallebene). Worauf dabei zu achten ist – besonders bei `sleep`/Timern und Nebenläufigkeit – steht unter [Refactoring & Migration](../Software-Konzepte/Refactoring%20%26%20Migration.md).

### Verbreitete openHAB-Entwurfsmuster

Die openHAB-Community hat eigene **Design Patterns** für wiederkehrende Regel-Probleme etabliert. Sie sind unabhängig von den klassischen [GoF-Mustern](../Design%20Pattern/README.md) und lohnen sich zu kennen:

| Muster | Zweck |
| --- | --- |
| **Time of Day** | Tagesabschnitt (morgen/tag/abend/nacht) als ein String-Item, statt Uhrzeiten in jeder Regel |
| **Separation of Behaviors** | Auslöser und Aktion trennen: Sensoren setzen einen Zustand, eine zentrale Regel reagiert |
| **Proxy Item / Unbound Item** | Virtuelles Item als Schaltpunkt für Szenen und Logik |
| **Gate Keeper** | Befehle entzerren (Mindestabstand), um Geräte/Busse nicht zu überlasten |
| **Debounce** | Prellende Sensoren entstören (erst nach Ruhephase reagieren) |
| **Gruppenbasierte Regeln** | Eine Regel für alle Mitglieder einer Gruppe statt je Item |
| **Timer Management** | Timer sauber anlegen, verlängern und abbrechen |

Diese Muster sind im Laborprojekt **openHAB Design Patterns** auf Rules DSL, JavaScript und Python 3 übertragen.

---

## Oberflächen

| Oberfläche | Einsatz im Labor |
| --- | --- |
| **MainUI** | Schlanker Schnellzugriff (Morgenroutine, Ausgangszustand, Rollladen, Jalousien, Lampen) |
| **Sitemaps** | Thematisch getrennte Oberflächen pro Aufgabe |
| **Eigene HTML/JS-Dashboards** | Ansprechende, raumbezogene Dashboards (über die REST API; auf den Wand-Tablets im Kiosk-Modus) |
| **HABPanel / CometVisu** | Alternativen – im Labor zugunsten eigener Dashboards verworfen |

Dashboards greifen über die **REST API** (und SSE für Live-Updates) zu – siehe [HTTP & REST](../Netzwerk/HTTP%20%26%20REST.md) und die eigenen REST-Clients.

---

## Betrieb

* **Als Dienst** (systemd), Autostart aktiviert (→ [systemd-Services](../Linux%20%26%20Werkzeuge/systemd-Services.md)).
* **Abhängigkeiten:** MQTT-Broker vor openHAB starten; Dienste, die openHAB brauchen, danach.
* **HTTPS** über einen Reverse Proxy, Zugriff nur verschlüsselt (→ [Zertifikate](../Best%20Practices/Zertifikate.md), [Web-Server & Deployment](../Best%20Practices/Web-Server%20%26%20Deployment.md)).
* **Konsole (Karaf):** erreichbar über `ssh -p 8101 openhab@localhost`. **Standardpasswort `habopen` ändern.**
* **Backup:** `conf/` per Git, zusätzlich `userdata` sichern; vor Updates Snapshot/Backup (→ [Backup-Strategien](../Backup-Strategien/README.md)).
* **Updates:** Release Notes lesen (Breaking Changes zwischen Hauptversionen), testen, dann produktiv – am besten zuerst in einer Testinstanz/VM.

---

## Sicherheit

* **API-Token** statt Basic Auth; Authentifizierung aktiviert lassen (keine Workarounds, die sie abschalten).
* Zugriff nur über **HTTPS** und, wo möglich, nur aus dem Labornetz.
* **Exec Binding** streng über die Whitelist begrenzen (→ [Exec Binding & Remote-Ausführung](../Best%20Practices/Exec-Binding%20%26%20Remote-Ausführung.md)).
* **MQTT** mit Passwort, TLS und ACL (→ [MQTT](../Best%20Practices/MQTT.md)).
* Roboter und unsichere Geräte in ein **getrenntes Netz** (VLAN).

---

## Checkliste für ein neues Gerät

* [ ] Feste IP per DHCP-Reservierung, Hostname (→ [DHCP](../Best%20Practices/DHCP.md))
* [ ] Zugriff abgesichert (SSH-Key, Zertifikate, MQTT-Passwort)
* [ ] Thing **als Textdatei** angelegt, Items mit Namensschema
* [ ] Items ins **semantische Modell** eingeordnet
* [ ] Bei Bedarf Persistenz-Strategie ergänzt
* [ ] In Dashboard/Sitemap aufgenommen
* [ ] Dokumentiert und ins Backup/Ansible-Inventar übernommen

---
