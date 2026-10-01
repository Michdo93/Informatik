# 🏗️ Skeleton, Boilerplate & Scaffolding

Drei Begriffe aus dem Bauwesen und dem Druckgewerbe, die alle mit dem **Anfang eines Projekts** zu tun haben – und die oft durcheinandergeworfen werden.

<!-- TOC -->
## Inhaltsverzeichnis

- [Auf einen Blick](#auf-einen-blick)
- [Skeleton](#skeleton)
  - [Herkunft](#herkunft)
  - [Ein gutes Projekt-Skeleton](#ein-gutes-projekt-skeleton)
- [Boilerplate](#boilerplate)
  - [Herkunft](#herkunft-1)
  - [Beispiele](#beispiele)
  - [Boilerplate reduzieren](#boilerplate-reduzieren)
- [Scaffolding](#scaffolding)
  - [Herkunft](#herkunft-2)
  - [Werkzeuge](#werkzeuge)
- [Abgrenzung: Scaffolding im Lehrkontext](#abgrenzung-scaffolding-im-lehrkontext)
- [Gute Praxis](#gute-praxis)
<!-- /TOC -->

## Auf einen Blick

| Begriff | Was ist es? | Analogie | Beispiel |
| --- | --- | --- | --- |
| **Skeleton** | Minimales, **lauffähiges** Grundgerüst eines Projekts | Rohbau / Skelett | Leeres Flask-Projekt mit `app.py`, `requirements.txt`, Tests |
| **Boilerplate** | Wiederkehrender **Pflichtcode** ohne eigene Logik | Standardklauseln im Vertrag | `if __name__ == "__main__":`, Getter/Setter in Java, Lizenzheader |
| **Scaffolding** | **Werkzeug/Vorgang**, das Gerüste oder Code generiert | Baugerüst | `cookiecutter`, `ansible-galaxy init`, `rails generate` |
| **Template** | Vorlage mit Platzhaltern, aus der das Skeleton erzeugt wird | Schablone | Cookiecutter-Template (→ [Templates & Templating](Templates%20%26%20Templating.md)) |
| **Starter Kit** | Skeleton + bereits ausgewählte Bibliotheken + Konfiguration | Fertighaus-Bausatz | „React + TypeScript + ESLint Starter“ |

---

## Skeleton

### Herkunft

Wie das **Skelett** eines Körpers: Es trägt alles, ist aber noch kein vollständiger Organismus. Verwandt ist der Begriff **Walking Skeleton** (Alistair Cockburn): eine **winzige End-to-End-Implementierung**, die alle Hauptkomponenten bereits verbindet – z. B. Sensor → MQTT → openHAB → Dashboard, auch wenn jede Station nur einen Dummy-Wert durchreicht.

### Ein gutes Projekt-Skeleton

```text
my-lab-project/
├── README.md                 # what, why, install, run (English)
├── LICENSE
├── .gitignore
├── .editorconfig
├── pyproject.toml            # dependencies, tool config (ruff, pytest)
├── src/
│   └── my_lab_project/
│       ├── __init__.py
│       └── __main__.py       # python -m my_lab_project
├── tests/
│   └── test_smoke.py         # at least one test that runs
├── config/
│   └── example.yaml          # example config, never real secrets
├── docs/
└── scripts/
    └── bootstrap.sh          # set up venv, install deps
```

Ein Skeleton ist **erst dann gut**, wenn es direkt nach dem Erzeugen

* [ ] fehlerfrei startet,
* [ ] einen Test ausführt, der grün ist,
* [ ] eine README hat, die erklärt, wie man es startet (→ [Best Practice Dokumentation](../Best%20Practices/Dokumentation.md)),
* [ ] Formatierung und Linting bereits konfiguriert hat (→ [Code-Formatierung](../Code-Formatierung/README.md)).

---

## Boilerplate

### Herkunft

Im 19. Jahrhundert lieferten Nachrichtenagenturen in den USA Texte (z. B. Anzeigen, Standardartikel) an Lokalzeitungen als fertig gegossene **Druckplatten aus Stahl** – sie erinnerten an das Blech, aus dem **Dampfkessel (boiler)** genietet wurden. Diese Texte wurden **unverändert** übernommen. Später übertrug man den Begriff auf Standardklauseln in Verträgen und schließlich auf Code.

### Beispiele

```java
// Java: much boilerplate for a simple data class (before records)
public class Sensor {
    private final String name;
    private final double value;

    public Sensor(String name, double value) { this.name = name; this.value = value; }
    public String getName() { return name; }
    public double getValue() { return value; }
    // equals(), hashCode(), toString() ...
}

// Java 16+: the same as a record
public record Sensor(String name, double value) { }
```

```python
# Python: dataclass removes boilerplate
from dataclasses import dataclass

@dataclass
class Sensor:
    name: str
    value: float
```

### Boilerplate reduzieren

* Sprachmittel nutzen: `@dataclass`, Java `record`, C# `record`, Kotlin `data class`.
* Generatoren/Annotations (Lombok), IDE-Live-Templates.
* Gemeinsamen Code in **Bibliotheken** oder **Basisklassen** auslagern.
* **Aber:** Boilerplate ist nicht immer schlecht – expliziter Code ist oft leichter zu verstehen als „Magie“.

---

## Scaffolding

### Herkunft

Engl. *scaffold* = **Baugerüst**. Das Gerüst wird aufgebaut, damit man das eigentliche Gebäude errichten kann – und später wieder abgebaut. Ruby on Rails machte den Begriff um 2005 populär: `rails generate scaffold` erzeugte aus einem Modell automatisch Datenbanktabelle, Controller und Views.

### Werkzeuge

| Werkzeug | Erzeugt | Aufruf |
| --- | --- | --- |
| **Cookiecutter** | Beliebige Projekte aus Templates | `cookiecutter gh:org/template` |
| **Copier** | Wie Cookiecutter, kann bestehende Projekte **aktualisieren** | `copier copy <template> <ziel>` |
| **ansible-galaxy** | Ansible-Rolle | `ansible-galaxy role init roles/mosquitto` |
| **ROS 2** | ROS-Paket | `ros2 pkg create --build-type ament_python my_pkg` |
| **npm / Vite** | Web-Projekt | `npm create vite@latest` |
| **dotnet** | C#-Projekt | `dotnet new console -n MyApp` |
| **Maven / Gradle** | Java-Projekt | `mvn archetype:generate`, `gradle init` |
| **Django** | Projekt / App | `django-admin startproject`, `python manage.py startapp` |
| **GitHub** | Repo aus Template-Repo | „Use this template“ |

**Beispiel – Ansible-Rolle:**

```bash
ansible-galaxy role init roles/mosquitto
```

```text
roles/mosquitto/
├── defaults/main.yml     # default variables (lowest priority)
├── files/
├── handlers/main.yml
├── meta/main.yml
├── tasks/main.yml
├── templates/            # *.j2 files
├── tests/
└── vars/main.yml
```

Gerade bei Ansible erzwingt das Scaffold eine **einheitliche Struktur** und macht Rollen wiederverwendbar (→ [Best Practice Ansible](../Best%20Practices/Ansible.md)).

---

## Abgrenzung: Scaffolding im Lehrkontext

In der Pädagogik bedeutet **Scaffolding** etwas anderes: Lernende bekommen am Anfang **viel Unterstützung** (Lückentexte, vorgegebener Code), die schrittweise abgebaut wird. Viele Laboraufgaben arbeiten genau so – mit einem vorgegebenen **Skeleton**, in dem nur die `TODO`-Stellen ausgefüllt werden.

---

## Gute Praxis

* [ ] Für wiederkehrende Projekttypen im Labor **ein eigenes Template** pflegen (z. B. „Python-Gerätetreiber mit MQTT“).
* [ ] Generierten Code **sofort committen**, bevor man ihn verändert – so sieht man im Diff, was man selbst geändert hat.
* [ ] Nicht benötigte generierte Dateien **löschen** statt leer liegen lassen.
* [ ] Das Template versionieren und in der README des erzeugten Projekts vermerken, aus welcher Template-Version es stammt.

---
