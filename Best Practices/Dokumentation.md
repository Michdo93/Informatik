# 📝 Best Practice: Dokumentation

Dieses Dokument regelt **nicht**, wie Kommentare im Code aussehen (dafür siehe [Code-Formatierung → Kommentare & Dokumentation](../Code-Formatierung/Kommentare%20&%20Dokumentation.md)), sondern **wie die Projektdokumentation** eines Programms, Repositories oder Systems aufgebaut sein muss.

Kurz gesagt: **Eine gute Dokumentation ist vollständig.** Wer das Repository zum ersten Mal sieht, muss das Programm ohne Rückfrage installieren, konfigurieren, starten und benutzen können.

<!-- TOC -->
## Inhaltsverzeichnis

- [Grundregeln](#grundregeln)
- [Was in eine vollständige Dokumentation gehört](#was-in-eine-vollständige-dokumentation-gehört)
- [README-Vorlage](#readme-vorlage)
- [Weitere Dokumente im Repository](#weitere-dokumente-im-repository)
- [Typische Fehler](#typische-fehler)
<!-- /TOC -->

## Grundregeln

| Regel | Begründung |
| --- | --- |
| **Sprache: Englisch** | Code, Issues, Commits und Doku in einer Sprache. Englisch erreicht alle (internationale Studierende, Open Source, spätere Arbeitgeber). *Dieses Kompendium ist bewusst eine Ausnahme: Es ist ein deutschsprachiges Nachschlagewerk.* |
| **Format: Markdown** | Wird von GitHub/GitLab direkt gerendert, ist versionierbar, diff-bar und braucht kein Spezialprogramm. Keine Word-/PDF-Dateien als Primärdoku. |
| **Ort: im Repository** | Doku liegt neben dem Code (`README.md`, `docs/`) und wird mit ihm versioniert. Ein Wiki oder eine Cloud-Datei veraltet schneller. |
| **Vollständigkeit** | Alles, was man zum Betrieb braucht, steht drin – nicht „das weiß man doch“. |
| **Aktualität** | Code-Änderung ohne Doku-Änderung ist eine unvollständige Änderung. Gehört in denselben Commit / Pull Request. |
| **Reproduzierbarkeit** | Befehle sind kopierbar (Codeblöcke), Versionen sind angegeben, Beispielkonfigurationen funktionieren. |

---

## Was in eine vollständige Dokumentation gehört

Die folgende Checkliste ist der **Mindestumfang**. Kapitel, die für ein Projekt nicht zutreffen, werden mit einem Satz begründet weggelassen („This tool has no configuration file.“), nicht stillschweigend.

| Bereich | Inhalt |
| --- | --- |
| **Overview** | Was macht das Programm? Für wen? Welches Problem löst es? Ein Satz + ein Absatz. |
| **Features** | Liste aller Funktionen. |
| **Requirements** | Betriebssystem, Hardware, Sprachversion, Abhängigkeiten (mit Versionen), benötigte Dienste (z. B. MQTT-Broker, Datenbank). |
| **Installation** | Alle Schritte, vom leeren System bis zum lauffähigen Programm. Inkl. Paketinstallation, virtueller Umgebung, Rechte, Dienste/`systemd`-Unit. |
| **Configuration** | **Alle** Konfigurationsmöglichkeiten: Datei(en), Umgebungsvariablen, CLI-Parameter. Je Option: Name, Typ, Default, Bedeutung, Beispiel. Am besten als Tabelle. |
| **Usage / Running** | Wie startet man es? Beispiele für typische Aufrufe. Ausgabe/Ergebnis. |
| **Commissioning (Inbetriebnahme)** | Was muss nach der Installation einmalig passieren? (Kalibrierung, Pairing, Zertifikate, erster Login, Firewall-Freigabe, Eintrag in Ansible …) |
| **Interfaces / API** | Alle Schnittstellen: REST-Endpunkte, MQTT-Topics und Payloads, CLI, Bibliotheks-API, Ports, Dateiformate. |
| **Architecture** | Komponenten und deren Zusammenspiel, idealerweise mit Diagramm (Mermaid, PlantUML). |
| **Troubleshooting** | Bekannte Fehler und ihre Lösung, wo die Logs liegen, wie man Debug-Ausgaben aktiviert. |
| **Development** | Wie baut, testet und debuggt man das Projekt? Coding Style, Branching. |
| **Changelog** | Was hat sich in welcher Version geändert (`CHANGELOG.md`, siehe [Keep a Changelog](https://keepachangelog.com/)). |
| **License** | Lizenz (`LICENSE`-Datei). |
| **Authors / Contact** | Wer ist verantwortlich? |

---

## README-Vorlage

Diese Vorlage kann direkt kopiert werden:

````markdown
# Project Name

Short description in one sentence.

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Interfaces](#interfaces)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Overview
What does it do, for whom and why?

## Features
- Feature A
- Feature B

## Requirements
- Ubuntu 22.04 / Raspberry Pi OS (Bookworm)
- Python >= 3.10
- Mosquitto MQTT broker >= 2.0

## Installation
```bash
git clone https://github.com/<user>/<repo>.git
cd <repo>
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Configuration
Configuration is read from `config.yaml`.

| Option        | Type   | Default     | Description                  |
| ------------- | ------ | ----------- | ---------------------------- |
| `mqtt.host`   | string | `localhost` | Hostname of the MQTT broker  |
| `mqtt.port`   | int    | `8883`      | TLS port of the MQTT broker  |
| `log_level`   | string | `INFO`      | `DEBUG`, `INFO`, `WARNING`   |

## Usage
```bash
python3 -m mytool --config config.yaml
```

## Interfaces
| Topic                     | Direction | Payload        | Description        |
| ------------------------- | --------- | -------------- | ------------------ |
| `device/power/set`        | in        | `ON` / `OFF`   | Switch the device  |
| `device/power/state`      | out       | `ON` / `OFF`   | Current state      |

## Troubleshooting
Logs: `journalctl -u mytool -f`

## License
MIT
````

---

## Weitere Dokumente im Repository

| Datei / Ordner | Zweck |
| --- | --- |
| `README.md` | Einstiegspunkt, immer vorhanden |
| `docs/` | Ausführliche Doku, wenn das README zu lang wird (z. B. `docs/configuration.md`, `docs/api.md`) |
| `CHANGELOG.md` | Versionshistorie |
| `CONTRIBUTING.md` | Regeln für Beiträge (Stil, Branches, Tests) |
| `LICENSE` | Lizenztext |
| `examples/` | Lauffähige Beispielkonfigurationen und -aufrufe |
| `.env.example` / `config.example.yaml` | Beispielkonfiguration **ohne** echte Passwörter |

Für große Projekte: Doku-Generatoren wie **MkDocs**, **Sphinx** oder **Docusaurus**, die aus Markdown eine Webseite bauen.

---

## Typische Fehler

* **„Siehe Code“** als Dokumentation – der Code erklärt *wie*, aber nicht *warum* und nicht *wie man es bedient*.
* Installationsanleitung, die nur auf dem eigenen Rechner funktioniert (fehlende Pakete, absolute Pfade wie `/home/max/...`).
* Konfigurationsoptionen, die nur im Quellcode auftauchen.
* Echte Passwörter, Tokens oder IP-Adressen interner Systeme in Beispielen.
* Screenshots statt kopierbarer Befehle.
* Doku, die nach der Abgabe der Abschlussarbeit niemand mehr anpasst. **Test:** Eine zweite Person installiert das Projekt nur mit Hilfe der Doku auf einem frischen System.

---
