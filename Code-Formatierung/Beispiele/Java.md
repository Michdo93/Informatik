# ☕ Java Styleguide (Beispiel)

Für Java gibt es die historischen **Oracle (Sun) Code Conventions** von 1997 und den heute meist verwendeten **[Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)**. Beide sind sich in den Grundzügen einig. Dieses Beispiel folgt dem Google Java Style Guide.

<!-- TOC -->
## Inhaltsverzeichnis

- [1. Allgemeines](#1-allgemeines)
- [2. Einrückung & Formatierung](#2-einrückung--formatierung)
- [3. Namenskonventionen](#3-namenskonventionen)
- [4. Dateien & Imports](#4-dateien--imports)
- [5. Kommentare & Dokumentation](#5-kommentare--dokumentation)
- [6. Fehlerbehandlung](#6-fehlerbehandlung)
- [7. Logging](#7-logging)
- [8. Sonstiges](#8-sonstiges)
<!-- /TOC -->

## 1. **Allgemeines**

* Sprache: Java 17 oder 21 (LTS-Versionen)
* Encoding: UTF-8
* Style basiert auf dem Google Java Style Guide
* Build: Maven oder Gradle (Projektstruktur siehe [Dateistruktur & Projektorganisation](../Dateistruktur%20&%20Projektorganisation.md))
* Formatierung mit `google-java-format` oder Spotless, Prüfung mit Checkstyle

---

## 2. **Einrückung & Formatierung**

* 2 Leerzeichen (Google) bzw. 4 Leerzeichen (Oracle, viele IDE-Defaults) – im Projekt einheitlich
* Fortsetzungszeilen: mindestens +4 Leerzeichen
* Max. Zeilenlänge: 100 Zeichen
* Klammerstil **K&R** („Egyptian Brackets“): öffnende Klammer am Zeilenende
* Geschweifte Klammern immer, auch bei einzeiligen `if`-Blöcken

  ```java
  public class TemperatureSensor {
    private final int channel;

    public TemperatureSensor(int channel) {
      this.channel = channel;
    }

    public double readCelsius() {
      if (channel < 0) {
        throw new IllegalStateException("Sensor not initialized");
      }
      return 21.5;
    }
  }
  ```

---

## 3. **Namenskonventionen**

| Element                      | Konvention              | Beispiel                   |
| ---------------------------- | ----------------------- | -------------------------- |
| Packages                     | alles klein, ohne `_`   | `de.hfu.smarthome.sensors` |
| Klassen, Interfaces, Enums   | `PascalCase`            | `TemperatureSensor`        |
| Methoden                     | `camelCase` (Verb)      | `readCelsius()`            |
| Variablen, Parameter, Felder | `camelCase`             | `sampleCount`              |
| Konstanten (`static final`)  | `SCREAMING_SNAKE_CASE`  | `MAX_RETRIES`              |
| Typparameter                 | Ein Großbuchstabe       | `T`, `E`, `K`, `V`         |
| Testklassen                  | Klassenname + `Test`    | `TemperatureSensorTest`    |

* Interfaces bekommen **kein** `I`-Präfix (anders als in C#)
* Package-Namen beginnen mit der umgekehrten Domain (`de.hfu...`)

---

## 4. **Dateien & Imports**

* Eine öffentliche Top-Level-Klasse pro Datei, Dateiname = Klassenname
* Reihenfolge in der Datei: Lizenz-Header, `package`, Imports, Klasse
* **Keine Wildcard-Imports** (`import java.util.*;`)
* Statische Imports in einem eigenen Block vor den normalen Imports

---

## 5. **Kommentare & Dokumentation**

* **Javadoc** für jede `public`- und `protected`-Klasse und -Methode
* Erster Satz ist eine Zusammenfassung (wird in Übersichten angezeigt)

  ```java
  /**
   * Reads the current temperature from the sensor.
   *
   * @return the temperature in degrees Celsius
   * @throws IllegalStateException if the sensor is not initialized
   */
  public double readCelsius() { ... }
  ```

---

## 6. **Fehlerbehandlung**

* Spezifische Exceptions fangen, niemals leere `catch`-Blöcke
* Checked Exceptions für erwartbare, behebbare Fehler (z. B. `IOException`), Unchecked (`RuntimeException`) für Programmierfehler
* Ressourcen mit **try-with-resources** schließen:

  ```java
  try (BufferedReader reader = Files.newBufferedReader(path)) {
    return reader.readLine();
  } catch (IOException e) {
    log.error("Could not read config file {}", path, e);
    throw new ConfigException("Config not readable", e);
  }
  ```

* Beim Weiterwerfen die Ursache (`cause`) mitgeben

---

## 7. **Logging**

* SLF4J als Fassade mit Logback oder Log4j2 als Implementierung
* Kein `System.out.println` und kein `e.printStackTrace()` im Produktivcode
* Platzhalter `{}` statt String-Verkettung:

  ```java
  private static final Logger log = LoggerFactory.getLogger(TemperatureSensor.class);

  log.info("Channel {} returned {} °C", channel, value);
  ```

---

## 8. **Sonstiges**

* `@Override` immer angeben
* Felder möglichst `private final`; Unveränderlichkeit bevorzugen (`record` ab Java 16)
* `equals()` und `hashCode()` immer gemeinsam überschreiben
* `Optional` als Rückgabewert statt `null`, aber nicht als Feld oder Parameter
* Strings mit `equals()` vergleichen, nicht mit `==`
* Tests mit JUnit 5 unter `src/test/java`

---
