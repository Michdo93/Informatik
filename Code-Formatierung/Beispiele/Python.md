# 🐍 Python Styleguide (Beispiel)

## 1. **Allgemeines**

* Sprache: Python 3.x
* Encoding: UTF-8
* Style basiert auf [PEP 8](https://peps.python.org/pep-0008/)

---

## 2. **Einrückung & Formatierung**

* 4 Leerzeichen pro Einrückung (kein Tab)
* Max. Zeilenlänge: 79 Zeichen (docstrings 72)
* Leerzeilen:

  * 2 Leerzeilen vor Klassen- und Funktionsdefinitionen auf Modulebene
  * 1 Leerzeile innerhalb von Klassenmethoden zur Strukturierung
* Keine Leerzeichen direkt vor `(`, `[` oder `{`
* Umgebrochene lange Zeilen mit Hängender Einrückung:

  ```python
  result = some_function_that_takes_parameters(
      param1, param2, param3,
      param4
  )
  ```

---

## 3. **Kommentare & Docstrings**

* Inline-Kommentare mit `#`, mindestens 2 Leerzeichen Abstand zum Code
* Mehrzeilige Kommentare mit mehreren `#`-Zeilen
* Docstrings in """Triple Quotes""" nach PEP 257
* Docstrings verwenden Google oder NumPy Style oder reStructuredText (RST)
* Beispiel Docstring (Google Style):

  ```python
  def add(a: int, b: int) -> int:
      """
      Add two numbers.

      Args:
          a (int): First number.
          b (int): Second number.

      Returns:
          int: The sum of a and b.
      """
      return a + b
  ```

---

## 4. **Namenkonventionen**

* Funktionen, Variablen: `lower_case_with_underscores`
* Klassen: `UpperCamelCase`
* Konstanten: `ALL_CAPS_WITH_UNDERSCORES`
* Module und Packages: kurze, kleingeschriebene Namen, keine Sonderzeichen

---

## 5. **Imports**

* Imports immer am Anfang der Datei
* Standardbibliothek, Drittanbieter, eigene Module in getrennten Blöcken
* Jeweils alphabetisch sortiert
* Beispiel:

  ```python
  import os
  import sys

  import requests

  import mypackage.utils
  ```

---

## 6. **Fehlerbehandlung**

* Nur spezifische Exceptions abfangen (nicht `except:`)
* Fehler protokollieren oder weiterreichen
* Eigene Exceptionklassen für spezifische Fehler

---

## 7. **Logging**

* Verwendung des `logging`-Moduls, kein `print` für Debugging
* Log-Level: DEBUG, INFO, WARNING, ERROR, CRITICAL

---

## 8. **Sonstiges**

* Verwende `with`-Statements für Ressourcenmanagement (`open()`, DB-Verbindungen)
* Vermeide globale Variablen
* Funktionen max. ca. 20-30 Zeilen lang
* Tests in separatem `tests/`-Ordner

---
