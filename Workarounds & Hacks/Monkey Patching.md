# 🐒 Monkey Patching

**Monkey Patching** bedeutet, **fremden Code zur Laufzeit zu verändern** – Funktionen, Methoden oder Klassen einer Bibliothek werden ersetzt oder ergänzt, **ohne deren Quellcode anzufassen**.

<!-- TOC -->
## Inhaltsverzeichnis

- [Herkunft des Namens](#herkunft-des-namens)
- [Beispiel in Python](#beispiel-in-python)
- [Legitimer Einsatz: Tests](#legitimer-einsatz-tests)
- [Andere Sprachen](#andere-sprachen)
- [Risiken](#risiken)
- [Regeln](#regeln)
<!-- /TOC -->

## Herkunft des Namens

Die verbreitete Erklärung: Zuerst sprach man (in der Zope/Python-Community, ca. 2000er) von **„Guerrilla Patch“** – ein heimlicher Patch, der zur Laufzeit „aus dem Hinterhalt“ zuschlägt. Aus *guerrilla* wurde durch Wortspiel *gorilla* (gleich ausgesprochen) und daraus – weil es oft gar nicht so mächtig ist – der kleinere **Monkey**. Gleichzeitig schwingt „monkeying around“ (herumpfuschen) mit.

---

## Beispiel in Python

```python
import json
import time

# Library function we want to change behaviour of
original_dumps = json.dumps


def dumps_sorted(obj, *args, **kwargs):
    kwargs.setdefault("sort_keys", True)
    return original_dumps(obj, *args, **kwargs)


json.dumps = dumps_sorted              # monkey patch: affects EVERY caller in the process
print(json.dumps({"b": 1, "a": 2}))    # {"a": 2, "b": 1}
json.dumps = original_dumps            # restore
```

Ein typischer Workaround-Fall: Eine Bibliothek hat einen Bug, der Fix ist noch nicht veröffentlicht:

```python
import somelib

_original = somelib.Client.reconnect

def _patched_reconnect(self, *args, **kwargs):
    # WORKAROUND: somelib <= 2.3 forgets to reset the backoff, see https://github.com/.../issues/123
    self._backoff = 1
    return _original(self, *args, **kwargs)

somelib.Client.reconnect = _patched_reconnect
```

---

## Legitimer Einsatz: Tests

In Tests ist Monkey Patching **Standard**, um externe Abhängigkeiten (Netzwerk, Zeit, Hardware) zu ersetzen:

```python
# test_heating.py - run with: pytest
import time


def is_night() -> bool:
    return not 6 <= time.localtime().tm_hour < 22


def test_is_night(monkeypatch):
    fake = time.struct_time((2026, 1, 1, 23, 0, 0, 3, 1, 0))
    monkeypatch.setattr(time, "localtime", lambda: fake)   # restored automatically
    assert is_night()
```

* **pytest:** `monkeypatch`-Fixture (setzt nach dem Test alles zurück)
* **unittest:** `unittest.mock.patch()` als Decorator oder Context Manager

---

## Andere Sprachen

| Sprache | Möglichkeit |
| --- | --- |
| **JavaScript** | Prototypen ändern: `Array.prototype.foo = …` – Grundlage von [Polyfills](Polyfill%20%26%20Shim.md) |
| **Ruby** | „Open Classes“: jede Klasse kann wieder geöffnet werden; *Refinements* begrenzen den Scope |
| **Java / C#** | Nicht direkt; über Bytecode-Manipulation (Java Agents, Mockito, Harmony für .NET) |
| **C/C++** | `LD_PRELOAD` unter Linux: eigene Bibliothek wird vor der echten geladen und überschreibt Funktionen |

```bash
# LD_PRELOAD: replace a libc function for a single program (debugging, workarounds)
LD_PRELOAD=./libfaketime.so.1 FAKETIME="2030-01-01 00:00:00" date
```

---

## Risiken

| Risiko | Erklärung |
| --- | --- |
| **Globale Wirkung** | Der Patch betrifft **alle** Nutzer im Prozess – auch andere Bibliotheken |
| **Versionsbruch** | Ändert die Bibliothek ihre Interna, bricht der Patch still |
| **Unsichtbarkeit** | Wer den Code liest, sieht nicht, dass die Funktion verändert wurde |
| **Reihenfolge** | Wurde vor dem Patch schon `from lib import func` gemacht, zeigt die alte Referenz noch auf das Original |
| **Konflikte** | Zwei Bibliotheken patchen dieselbe Funktion |

---

## Regeln

* [ ] Nur, wenn es **keine Alternative** gibt (Konfiguration, Unterklasse, Wrapper/[Decorator](../Design%20Pattern/Strukturmuster/Decorator.md), Upstream-Fix).
* [ ] **An einer zentralen Stelle** patchen (z. B. `patches.py`), nicht verstreut.
* [ ] **Version prüfen** und Patch nur für betroffene Versionen anwenden.
* [ ] Kommentar mit **Begründung und Link** zum Bugreport.
* [ ] In Tests: immer mit `monkeypatch`/`mock.patch`, die automatisch zurücksetzen.

---
