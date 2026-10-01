# 🔧 C Styleguide (Beispiel)

Für C gibt es – anders als bei Python mit PEP 8 – **keinen einzigen offiziellen Styleguide**. Verbreitet sind vor allem der **Linux Kernel Coding Style**, der **GNU Coding Style** und für sicherheitskritische/Embedded-Software **Barr-C** bzw. **MISRA C**. Dieses Beispiel orientiert sich am **Linux Kernel Coding Style** mit einigen pragmatischen Abweichungen (Spaces statt Tabs), wie sie in vielen Embedded- und Hochschulprojekten üblich sind.

<!-- TOC -->
## Inhaltsverzeichnis

- [1. Allgemeines](#1-allgemeines)
- [2. Einrückung & Formatierung](#2-einrückung--formatierung)
- [3. Namenskonventionen](#3-namenskonventionen)
- [4. Header-Dateien](#4-header-dateien)
- [5. Kommentare & Dokumentation](#5-kommentare--dokumentation)
- [6. Fehlerbehandlung](#6-fehlerbehandlung)
- [7. Logging](#7-logging)
- [8. Sonstiges](#8-sonstiges)
<!-- /TOC -->

## 1. **Allgemeines**

* Sprachstandard: **C11** (oder C17), kompiliert mit `-std=c11 -Wall -Wextra -Wpedantic`
* Encoding: UTF-8, Zeilenenden LF
* Style angelehnt an den [Linux Kernel Coding Style](https://www.kernel.org/doc/html/latest/process/coding-style.html)
* Formatierung automatisiert mit `clang-format` (Datei `.clang-format` im Repo-Root)
* Für sicherheitskritischen Code zusätzlich: [Barr-C](https://barrgroup.com/embedded-systems/books/embedded-c-coding-standard) oder MISRA C

---

## 2. **Einrückung & Formatierung**

* 4 Leerzeichen pro Einrückung (Linux-Kernel selbst: Tabs mit Breite 8)
* Max. Zeilenlänge: 80–100 Zeichen
* Klammerstil **K&R**: öffnende Klammer bei Kontrollstrukturen in derselben Zeile, bei Funktionen in einer eigenen Zeile
* Immer geschweifte Klammern, auch bei Einzeilern (vermeidet Fehler wie Apples „goto fail“)
* Leerzeichen nach Schlüsselwörtern (`if (`, `for (`, `while (`), aber nicht nach Funktionsnamen (`foo(x)`)
* Der `*` gehört zum Variablennamen: `char *name;` (nicht `char* name;`)

  ```c
  int read_sensor(int channel)
  {
      if (channel < 0) {
          return -1;
      }

      for (int i = 0; i < MAX_RETRIES; i++) {
          do_something(i);
      }
      return 0;
  }
  ```

---

## 3. **Namenskonventionen**

* Funktionen und Variablen: `snake_case` (`read_sensor`, `buffer_len`)
* Makros und Konstanten (`#define`, `enum`-Werte): `SCREAMING_SNAKE_CASE`
* Typen per `typedef` nur sparsam und ohne Suffix `_t` (das ist von POSIX reserviert) – oft ist es klarer, direkt `struct sensor` zu schreiben
* Modulpräfix für alle öffentlichen Funktionen, da C keine Namespaces kennt: `mqtt_connect()`, `mqtt_publish()`
* Globale Variablen mit Präfix `g_` – oder besser: vermeiden

---

## 4. **Header-Dateien**

* Jede `.c`-Datei hat (sofern sie Funktionen nach außen anbietet) eine gleichnamige `.h`-Datei
* Include-Guards (oder `#pragma once`, nicht standardisiert, aber von allen gängigen Compilern unterstützt):

  ```c
  #ifndef SENSOR_H
  #define SENSOR_H

  #include <stdint.h>

  int sensor_init(uint8_t address);
  int sensor_read(uint8_t address, int16_t *value);

  #endif /* SENSOR_H */
  ```

* Reihenfolge der Includes: eigene Header zuerst, dann Bibliotheks-Header, dann Standardbibliothek
* `static` für alle Funktionen und Variablen, die nur innerhalb einer Datei gebraucht werden

---

## 5. **Kommentare & Dokumentation**

* Kommentare mit `/* ... */` oder `//` (ab C99 erlaubt)
* Öffentliche Funktionen mit **Doxygen**-Kommentar im Header:

  ```c
  /**
   * @brief Read a raw value from the sensor.
   *
   * @param address I2C address of the sensor.
   * @param value   Pointer that receives the measured value.
   * @return 0 on success, negative errno code on failure.
   */
  int sensor_read(uint8_t address, int16_t *value);
  ```

---

## 6. **Fehlerbehandlung**

* C hat keine Exceptions: Fehler werden über **Rückgabewerte** signalisiert (Konvention: `0` = Erfolg, negativ = Fehlercode, z. B. `-EINVAL`)
* Rückgabewerte von Systemaufrufen **immer** prüfen (`malloc`, `fopen`, `read`, …)
* Aufräumen per zentralem Ausgang mit `goto` ist im Linux-Kernel ausdrücklich erlaubt und üblich:

  ```c
  int load_config(const char *path)
  {
      int ret = -1;
      FILE *fp = fopen(path, "r");
      if (!fp) {
          return -errno;
      }

      char *buf = malloc(BUF_SIZE);
      if (!buf) {
          goto out_close;
      }

      ret = parse(fp, buf);

      free(buf);
  out_close:
      fclose(fp);
      return ret;
  }
  ```

---

## 7. **Logging**

* In Anwendungen: `syslog()` (POSIX) oder eine kleine Logging-Bibliothek wie `log.c` bzw. `zlog`
* Fehlermeldungen auf `stderr`, nie auf `stdout`
* Auf Mikrocontrollern: Logging über UART, abschaltbar per Makro (`#ifdef DEBUG`)

---

## 8. **Sonstiges**

* Speicher, der mit `malloc` angefordert wird, wird von derselben Komponente mit `free` freigegeben
* Keine „magischen Zahlen“ – stattdessen `#define` oder `enum`
* Feste Integer-Breiten aus `<stdint.h>` verwenden (`uint8_t`, `int32_t`), besonders bei Hardware und Protokollen
* `const` konsequent für Parameter, die nicht verändert werden
* Statische Analyse: `cppcheck`, `clang-tidy`, Sanitizer (`-fsanitize=address,undefined`)
* Beispiel `.clang-format`:

  ```yaml
  BasedOnStyle: LLVM
  IndentWidth: 4
  ColumnLimit: 100
  BreakBeforeBraces: Linux
  PointerAlignment: Right
  ```

---
