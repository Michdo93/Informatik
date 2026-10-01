# ➕ C++ Styleguide (Beispiel)

Für C++ gibt es mehrere etablierte Styleguides. Die wichtigsten sind der **[Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)**, der **LLVM Coding Standard** und die **[C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/)** von Bjarne Stroustrup und Herb Sutter. Die Core Guidelines regeln vor allem **wie man modernes C++ sicher benutzt**, die anderen vor allem **wie Code aussieht**. Dieses Beispiel kombiniert beides. In der ROS-Welt wird übrigens ebenfalls ein an Google angelehnter Stil verwendet ([ROS 2 Code Style](https://docs.ros.org/en/rolling/The-ROS2-Project/Contributing/Code-Style-Language-Versions.html)).

<!-- TOC -->
## Inhaltsverzeichnis

- [1. Allgemeines](#1-allgemeines)
- [2. Einrückung & Formatierung](#2-einrückung--formatierung)
- [3. Namenskonventionen](#3-namenskonventionen)
- [4. Header & Includes](#4-header--includes)
- [5. Kommentare & Dokumentation](#5-kommentare--dokumentation)
- [6. Speicher & Ressourcen (modernes C++)](#6-speicher--ressourcen-modernes-c)
- [7. Fehlerbehandlung & Logging](#7-fehlerbehandlung--logging)
- [8. Sonstiges](#8-sonstiges)
<!-- /TOC -->

## 1. **Allgemeines**

* Sprachstandard: **C++17** (oder C++20), kompiliert mit `-Wall -Wextra -Wpedantic`
* Encoding: UTF-8, Zeilenenden LF
* Style basiert auf dem Google C++ Style Guide, Sprachnutzung nach den C++ Core Guidelines
* Formatierung automatisiert mit `clang-format`, statische Analyse mit `clang-tidy`
* Build-System: CMake

---

## 2. **Einrückung & Formatierung**

* 2 Leerzeichen (Google) oder 4 Leerzeichen (viele Teams, ROS 2) – **im Projekt einheitlich**
* Max. Zeilenlänge: 80 (Google) bzw. 100 Zeichen
* Öffnende Klammer in derselben Zeile (K&R/„Attach“)
* `public:`, `protected:`, `private:` um 1 Leerzeichen eingerückt (Google) – in dieser Reihenfolge

  ```cpp
  class TemperatureSensor {
   public:
    explicit TemperatureSensor(int channel);

    double ReadCelsius() const;

   private:
    int channel_;
  };
  ```

---

## 3. **Namenskonventionen**

| Element                 | Google-Stil              | Beispiel                 |
| ----------------------- | ------------------------ | ------------------------ |
| Klassen, Structs, Enums | `PascalCase`             | `TemperatureSensor`      |
| Funktionen, Methoden    | `PascalCase`             | `ReadCelsius()`          |
| Variablen               | `snake_case`             | `sample_count`           |
| Member-Variablen        | `snake_case` + `_`       | `channel_`               |
| Konstanten              | `k` + `PascalCase`       | `kMaxRetries`            |
| Namespaces              | `snake_case`             | `smart_home::sensors`    |
| Makros                  | `SCREAMING_SNAKE_CASE`   | `PROJECT_SENSOR_H_`      |
| Dateien                 | `snake_case.cpp` / `.h`  | `temperature_sensor.cpp` |

> Hinweis: Die Standardbibliothek selbst nutzt durchgehend `snake_case` (`std::vector::push_back`). Viele Teams übernehmen das auch für Methoden. Wichtig ist nur: **ein Stil pro Projekt**.

---

## 4. **Header & Includes**

* Header-Endung `.h` oder `.hpp`, Implementierung `.cpp`
* Include-Guards im Format `<PROJECT>_<PATH>_<FILE>_H_` oder `#pragma once`
* Include-Reihenfolge (Google): zugehöriger Header, C-System-Header, C++-Standardbibliothek, andere Bibliotheken, eigene Header – jeweils durch Leerzeile getrennt
* **Kein** `using namespace std;` in Header-Dateien

  ```cpp
  #include "sensors/temperature_sensor.h"

  #include <cstdint>
  #include <memory>
  #include <string>

  #include <rclcpp/rclcpp.hpp>

  #include "common/logging.h"
  ```

---

## 5. **Kommentare & Dokumentation**

* `//` für normale Kommentare
* Öffentliche Schnittstellen mit **Doxygen** (`/** ... */` oder `///`)

  ```cpp
  /// @brief Reads the current temperature.
  /// @return Temperature in degrees Celsius.
  /// @throws SensorError if the sensor does not respond.
  double ReadCelsius() const;
  ```

---

## 6. **Speicher & Ressourcen (modernes C++)**

* **RAII**: Ressourcen (Speicher, Dateien, Mutex, Sockets) werden im Konstruktor erworben und im Destruktor freigegeben
* Kein rohes `new`/`delete` – stattdessen `std::make_unique` / `std::make_shared`
* Besitzverhältnisse über Typen ausdrücken: `std::unique_ptr` = alleiniger Besitz, `std::shared_ptr` = geteilter Besitz, roher Zeiger/Referenz = nur Zugriff ohne Besitz
* `const` und `constexpr` wo immer möglich
* `auto`, wenn der Typ offensichtlich ist (`auto sensor = std::make_unique<TemperatureSensor>(1);`)
* `enum class` statt einfachem `enum`
* `nullptr` statt `NULL` oder `0`
* `override` bei überschriebenen virtuellen Methoden

---

## 7. **Fehlerbehandlung & Logging**

* Exceptions für echte Ausnahmefälle; eigene Exception-Klassen erben von `std::runtime_error` o. ä.
* Hinweis: Google verbietet Exceptions in eigenem Code (historische Gründe), in Embedded-Projekten sind sie oft per `-fno-exceptions` deaktiviert – dann Fehlercodes oder `std::optional` / `std::expected` (C++23)
* Logging: `spdlog`, in ROS `RCLCPP_INFO(...)`, nie `std::cout` für Fehler (stattdessen `std::cerr`)

  ```cpp
  class SensorError : public std::runtime_error {
   public:
    using std::runtime_error::runtime_error;
  };
  ```

---

## 8. **Sonstiges**

* Regel der Null / Drei / Fünf beachten: Entweder keine der Sonderfunktionen (Destruktor, Copy-/Move-Konstruktor, Copy-/Move-Zuweisung) selbst schreiben oder alle bewusst definieren
* Konstruktoren mit einem Parameter als `explicit` markieren
* Templates nur, wenn sie echten Mehrwert bringen (Lesbarkeit und Fehlermeldungen leiden)
* Beispiel `.clang-format`:

  ```yaml
  BasedOnStyle: Google
  IndentWidth: 2
  ColumnLimit: 100
  ```

---
