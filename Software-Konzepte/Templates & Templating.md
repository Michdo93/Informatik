# 📄 Templates & Templating

Ein **Template** (Vorlage, Schablone) ist ein Dokument mit **festen Teilen** und **Platzhaltern**. Beim **Templating** (auch *Rendering*) setzt eine **Template-Engine** konkrete Werte in die Platzhalter ein. Das Prinzip ist uralt – der **Serienbrief** in Word ist Templating.

<!-- TOC -->
## Inhaltsverzeichnis

- [Wortherkunft](#wortherkunft)
- [Was bedeutet „Template“ in der IT?](#was-bedeutet-template-in-der-it)
- [Text-Templating mit Jinja2](#text-templating-mit-jinja2)
- [HTML-Templates und Sicherheit](#html-templates-und-sicherheit)
- [Generics und C++-Templates](#generics-und-c-templates)
- [Gute Praxis](#gute-praxis)
<!-- /TOC -->

## Wortherkunft

*Template* stammt vermutlich von *templet*, einer **Holz- oder Metallschablone**, die Handwerker (Zimmerleute, Steinmetze) nutzten, um immer wieder dieselbe Form anzureißen – verwandt mit französisch *temple*, einem Teil am Webstuhl. Die Idee ist dieselbe wie in der IT: **eine** Form, **viele** gleichartige Ergebnisse.

---

## Was bedeutet „Template“ in der IT?

Das Wort wird für mehrere, deutlich verschiedene Dinge benutzt:

| Bedeutung | Beispiel | Ergebnis |
| --- | --- | --- |
| **Text-/Datei-Templates** | Jinja2, Ansible `template`-Modul, Mustache, Handlebars | Konfigurationsdatei, HTML, E-Mail |
| **HTML-/View-Templates** | Django Templates, Flask/Jinja2, Thymeleaf, Razor | Webseite |
| **Projekt-Templates** | GitHub Template Repository, Cookiecutter | Neues Projekt (→ [Skeleton & Scaffolding](Skeleton%2C%20Boilerplate%20%26%20Scaffolding.md)) |
| **Dokument-Templates** | LaTeX-Vorlage, Word `.dotx`, Markdown-Issue-Templates | Abschlussarbeit, Bericht |
| **VM-/Container-Templates** | Proxmox-VM-Template, Docker-Image als Vorlage | Neue VM/Container |
| **C++-Templates / Generics** | `std::vector<int>`, Java `List<String>` | Typspezifischer Code (zur Compile-Zeit) |
| **Template Method** | Design Pattern | Algorithmus-Skelett (→ [Template Method](../Design%20Pattern/Verhaltensmuster/Template%20Method.md)) |

---

## Text-Templating mit Jinja2

Jinja2 ist die Template-Engine von **Flask**, **Ansible**, **SaltStack**, **Home Assistant** und vielen weiteren Werkzeugen. Die Syntax lohnt sich also zu lernen.

| Syntax | Bedeutung |
| --- | --- |
| `{{ variable }}` | Ausdruck ausgeben |
| `{% if %} … {% endif %}` | Steuerung (Bedingung, Schleife) |
| `{# Kommentar #}` | Kommentar, erscheint nicht in der Ausgabe |
| `{{ name \| upper }}` | Filter anwenden |
| `{{ port \| default(1883) }}` | Standardwert |

**Template `mosquitto.conf.j2`:**

```jinja
# {{ ansible_managed | default("Managed by Ansible - do not edit by hand") }}
per_listener_settings true

listener {{ mqtt_port | default(8883) }}
cafile   /etc/mosquitto/certs/{{ inventory_hostname }}_ca.crt
certfile /etc/mosquitto/certs/{{ inventory_hostname }}.crt
keyfile  /etc/mosquitto/certs/{{ inventory_hostname }}.key
allow_anonymous false
password_file /etc/mosquitto/passwd
acl_file /etc/mosquitto/acl

{% for bridge in mqtt_bridges | default([]) %}
connection {{ bridge.name }}
address {{ bridge.host }}:{{ bridge.port }}
topic {{ bridge.topic }} both 0
{% endfor %}
```

**Rendern mit Python:**

```python
from jinja2 import Environment, StrictUndefined

env = Environment(undefined=StrictUndefined, trim_blocks=True, lstrip_blocks=True)
template = env.from_string("""\
[{{ group }}]
{% for host in hosts %}
{{ host.name }} ansible_host={{ host.ip }}
{% endfor %}
""")

print(template.render(group="raspberrypis",
                      hosts=[{"name": "pi-mqtt", "ip": "192.168.10.21"},
                             {"name": "pi-beamer", "ip": "192.168.10.22"}]))
```

> **Tipp:** `StrictUndefined` setzen. Sonst werden Tippfehler in Variablennamen stillschweigend zu leeren Strings – und die Konfigurationsdatei ist kaputt, ohne dass es auffällt.

**Mit Ansible:**

```yaml
- name: Deploy mosquitto configuration
  ansible.builtin.template:
    src: mosquitto.conf.j2
    dest: /etc/mosquitto/conf.d/lab.conf
    owner: root
    mode: "0644"
  notify: Restart mosquitto
```

---

## HTML-Templates und Sicherheit

Templates, die HTML erzeugen, müssen Benutzereingaben **escapen**. Sonst entsteht **Cross-Site Scripting (XSS)**:

```python
from jinja2 import Environment

unsafe = Environment(autoescape=False).from_string("<p>Hallo {{ name }}</p>")
safe = Environment(autoescape=True).from_string("<p>Hallo {{ name }}</p>")

payload = "<script>alert('xss')</script>"
print(unsafe.render(name=payload))   # script would run in the browser
print(safe.render(name=payload))     # &lt;script&gt;... is shown as text
```

**Server-Side Template Injection (SSTI):** Niemals Benutzereingaben **als Template** rendern (`env.from_string(user_input)`), sondern nur **als Variable** übergeben. Sonst können Angreifer Code auf dem Server ausführen.

---

## Generics und C++-Templates

In C++ erzeugt der Compiler aus einem Template für jeden verwendeten Typ eigenen Code:

```cpp
template <typename T>
T clamp_value(T value, T low, T high) {
    return value < low ? low : (value > high ? high : value);
}

int    a = clamp_value(120, 0, 100);        // instantiated for int
double b = clamp_value(1.7, 0.0, 1.0);      // instantiated for double
```

Java und C# nennen das **Generics** (`List<T>`), Python hat **Type Hints** mit `TypeVar` / `list[T]` – die aber nur für statische Prüfwerkzeuge (mypy) relevant sind.

---

## Gute Praxis

* [ ] Templates in **eigenen Dateien** (`templates/`, Endung `.j2`), nicht als lange Strings im Code.
* [ ] Am Anfang der erzeugten Datei vermerken, dass sie generiert ist („do not edit by hand“).
* [ ] **Logik minimieren:** Berechnungen gehören in den Code/die Variablen, nicht ins Template.
* [ ] Fehlende Variablen als Fehler behandeln (`StrictUndefined`).
* [ ] HTML immer mit **Autoescape**.
* [ ] Erzeugte Dateien **validieren** (z. B. `nginx -t`, `sshd -t -f <datei>`, im Ansible-`template`-Modul über `validate:`), bevor sie aktiv werden.
* [ ] Für jedes Template ein Beispiel der gerenderten Ausgabe in der Doku.

---
