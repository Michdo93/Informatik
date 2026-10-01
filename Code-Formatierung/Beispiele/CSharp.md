# #️⃣ C# Styleguide (Beispiel)

Für C# gibt es mit den **[Microsoft C# Coding Conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)** und den **[.NET Framework Design Guidelines](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/)** einen faktischen Standard. Fast alle Projekte folgen diesen, und Visual Studio, Rider und `dotnet format` setzen sie direkt um. Dateiname: `CSharp.md`, weil `#` in Dateinamen und URLs Probleme macht.

<!-- TOC -->
## Inhaltsverzeichnis

- [1. Allgemeines](#1-allgemeines)
- [2. Einrückung & Formatierung](#2-einrückung--formatierung)
- [3. Namenskonventionen](#3-namenskonventionen)
- [4. Dateien & Namespaces](#4-dateien--namespaces)
- [5. Kommentare & Dokumentation](#5-kommentare--dokumentation)
- [6. Fehlerbehandlung](#6-fehlerbehandlung)
- [7. Logging](#7-logging)
- [8. Sonstiges](#8-sonstiges)
<!-- /TOC -->

## 1. **Allgemeines**

* Sprache: C# 12 / .NET 8 (LTS) oder neuer
* Encoding: UTF-8, Zeilenenden: im Projekt einheitlich (per `.editorconfig` bzw. `.gitattributes` festlegen)
* Style basiert auf den Microsoft C# Coding Conventions
* Regeln werden über `.editorconfig` definiert und mit `dotnet format` durchgesetzt

---

## 2. **Einrückung & Formatierung**

* 4 Leerzeichen pro Einrückung (kein Tab)
* Max. Zeilenlänge: ca. 120 Zeichen
* Klammerstil **Allman**: geschweifte Klammern stehen **in einer eigenen Zeile**
* Eine Anweisung pro Zeile, eine Deklaration pro Zeile
* Leerzeile zwischen Methoden und Properties

  ```csharp
  public class TemperatureSensor
  {
      private readonly int _channel;

      public TemperatureSensor(int channel)
      {
          _channel = channel;
      }

      public double ReadCelsius()
      {
          if (_channel < 0)
          {
              throw new InvalidOperationException("Sensor not initialized.");
          }

          return 21.5;
      }
  }
  ```

---

## 3. **Namenskonventionen**

| Element                                   | Konvention                | Beispiel                |
| ----------------------------------------- | ------------------------- | ----------------------- |
| Klassen, Structs, Records, Enums          | `PascalCase`              | `TemperatureSensor`     |
| Interfaces                                | `I` + `PascalCase`        | `ISensor`               |
| Methoden, Properties, Events              | `PascalCase`              | `ReadCelsius()`, `Name` |
| Öffentliche Felder / Konstanten           | `PascalCase`              | `MaxRetries`            |
| Private Felder                            | `_camelCase`              | `_channel`              |
| Parameter, lokale Variablen               | `camelCase`               | `sampleCount`           |
| Asynchrone Methoden                       | Suffix `Async`            | `ReadCelsiusAsync()`    |
| Namespaces                                | `Firma.Produkt.Bereich`   | `Hfu.SmartHome.Sensors` |
| Generische Typparameter                   | `T` bzw. `T` + Name       | `T`, `TResult`          |

> Konstanten sind in C# **nicht** `SCREAMING_SNAKE_CASE`, sondern `PascalCase` – ein häufiger Fehler von Umsteigern aus C/Java.

---

## 4. **Dateien & Namespaces**

* Eine öffentliche Klasse pro Datei, Dateiname = Klassenname (`TemperatureSensor.cs`)
* Ordnerstruktur entspricht dem Namespace
* File-scoped Namespaces (ab C# 10) sparen eine Einrückungsebene:

  ```csharp
  namespace Hfu.SmartHome.Sensors;

  public class TemperatureSensor { /* ... */ }
  ```

* `using`-Direktiven außerhalb des Namespace, `System.*` zuerst

---

## 5. **Kommentare & Dokumentation**

* `//` für normale Kommentare, mit Leerzeichen nach `//`
* Öffentliche APIs mit **XML-Dokumentationskommentaren** (`///`), daraus erzeugen IDE und DocFX die Dokumentation

  ```csharp
  /// <summary>
  /// Reads the current temperature.
  /// </summary>
  /// <returns>The temperature in degrees Celsius.</returns>
  /// <exception cref="InvalidOperationException">Thrown if the sensor is not initialized.</exception>
  public double ReadCelsius() { /* ... */ }
  ```

---

## 6. **Fehlerbehandlung**

* Spezifische Exceptions fangen, nie leeres `catch { }`
* Eigene Exceptions enden auf `Exception` (`SensorException`)
* Zum Weiterwerfen `throw;` verwenden, **nicht** `throw ex;` (sonst geht der Stacktrace verloren)
* Ressourcen mit `using` freigeben:

  ```csharp
  using var stream = File.OpenRead(path);
  ```

---

## 7. **Logging**

* `Microsoft.Extensions.Logging` (`ILogger<T>`) per Dependency Injection, alternativ Serilog oder NLog
* Strukturierte Platzhalter statt String-Interpolation:

  ```csharp
  _logger.LogInformation("Sensor {Channel} returned {Value} °C", _channel, value);
  ```

---

## 8. **Sonstiges**

* `var`, wenn der Typ aus der rechten Seite offensichtlich ist
* Sprach-Schlüsselwörter statt Framework-Typen: `string`, `int` statt `String`, `Int32`
* `async`/`await` durchgängig verwenden, kein `.Result` oder `.Wait()` (Deadlock-Gefahr)
* Nullable Reference Types aktivieren (`<Nullable>enable</Nullable>`)
* Beispiel `.editorconfig`-Ausschnitt:

  ```ini
  [*.cs]
  indent_size = 4
  csharp_new_line_before_open_brace = all
  dotnet_naming_rule.private_fields_underscore.severity = warning
  dotnet_naming_rule.private_fields_underscore.symbols = private_fields
  dotnet_naming_rule.private_fields_underscore.style = underscore_camel
  dotnet_naming_symbols.private_fields.applicable_kinds = field
  dotnet_naming_symbols.private_fields.applicable_accessibilities = private
  dotnet_naming_style.underscore_camel.capitalization = camel_case
  dotnet_naming_style.underscore_camel.required_prefix = _
  ```

---
