# Fehlermeldungen, Logging & Exceptions

Auch für **Fehlermeldungen, Logging und Exceptions** gibt es **Standards und Best Practices**, oft sogar mit offiziellen Guidelines je nach Sprache oder Framework.

---

<!-- TOC -->
## Inhaltsverzeichnis

- [🛠️ 1. Fehlermeldungen (Error Messages)](#️-1-fehlermeldungen-error-messages)
  - [✅ Best Practices für Fehlermeldungen](#-best-practices-für-fehlermeldungen)
  - [🧾 Beispiel (für Frontend und Logging)](#-beispiel-für-frontend-und-logging)
- [🧱 2. Logging-Standards](#-2-logging-standards)
  - [📒 Gängige Log-Level (nach Syslog & RFC 5424)](#-gängige-log-level-nach-syslog--rfc-5424)
  - [🔧 Formate und Tools je nach Sprache](#-formate-und-tools-je-nach-sprache)
  - [🔄 Beispiel (strukturierter Log in JSON)](#-beispiel-strukturierter-log-in-json)
- [🚨 3. Exceptions & Fehlerbehandlung](#-3-exceptions--fehlerbehandlung)
  - [🔄 Prinzipien moderner Fehlerbehandlung](#-prinzipien-moderner-fehlerbehandlung)
  - [🧱 Beispiele: Eigene Fehlerklassen](#-beispiele-eigene-fehlerklassen)
- [🧾 Standards und Formate](#-standards-und-formate)
- [🔄 Optional: Logging-Integrationen](#-optional-logging-integrationen)
<!-- /TOC -->

## 🛠️ 1. **Fehlermeldungen (Error Messages)**

### ✅ **Best Practices für Fehlermeldungen**

| Prinzip                         | Beschreibung                                                               |
| ------------------------------- | -------------------------------------------------------------------------- |
| **Klar & verständlich**         | Menschlich lesbar, nicht nur für Entwickler.                               |
| **Konkret statt generisch**     | Nicht: `"Error 500"` — besser: `"Datenbankverbindung fehlgeschlagen"`      |
| **Kontext enthalten**           | Wer hat den Fehler verursacht, wo und warum?                               |
| **Keine sensiblen Infos**       | Keine Passwörter, Tokens, Pfade usw. anzeigen.                             |
| **Für Benutzer UND Entwickler** | Benutzer: „Datei konnte nicht geladen werden“, Entwickler-Log: Stacktrace. |

### 🧾 Beispiel (für Frontend und Logging)

```json
{
  "error": "User not found",
  "code": 404,
  "timestamp": "2025-05-30T15:23:00Z",
  "trace_id": "abc123xyz",
  "details": "No user found with ID 42"
}
```

---

## 🧱 2. **Logging-Standards**

### 📒 Gängige Log-Level (nach [Syslog & RFC 5424](https://datatracker.ietf.org/doc/html/rfc5424))

| Level   | Zweck                                                       |
| ------- | ----------------------------------------------------------- |
| `TRACE` | Detaillierteste Informationen (z. B. jeder Funktionsaufruf) |
| `DEBUG` | Für Entwickler – Debugging-Daten                            |
| `INFO`  | Normale Ereignisse (z. B. "User eingeloggt")                |
| `WARN`  | Unerwartet, aber nicht kritisch                             |
| `ERROR` | Fehler, der behandelt wird                                  |
| `FATAL` | Kritischer Fehler, der das System stoppt                    |

### 🔧 Formate und Tools je nach Sprache

| Sprache    | Logging-Framework / Standard      | Struktur/Format              |
| ---------- | --------------------------------- | ---------------------------- |
| Python     | `logging`, `structlog`, `loguru`  | JSON, Syslog, File           |
| Java       | `SLF4J`, `Logback`, `Log4j2`      | Pattern/Layout-Konfiguration |
| JavaScript | `winston`, `pino`, `console.log`  | JSON, stdout, rotating logs  |
| Go         | `log`, `logrus`, `zap`, `zerolog` | JSON/structured preferred    |
| PHP        | `Monolog`                         | PSR-3 konform                |
| C# (.NET)  | `Microsoft.Extensions.Logging`    | Seriell, structured logs     |
| Rust       | `log`, `env_logger`, `tracing`    | Structured, leveled          |

### 🔄 Beispiel (strukturierter Log in JSON)

```json
{
  "level": "ERROR",
  "timestamp": "2025-05-30T13:01:12Z",
  "message": "Login fehlgeschlagen",
  "user_id": 42,
  "ip": "192.168.1.10",
  "error_code": "AUTH_401"
}
```

---

## 🚨 3. **Exceptions & Fehlerbehandlung**

### 🔄 Prinzipien moderner Fehlerbehandlung

| Prinzip                        | Beispiel/Kommentar                            |
| ------------------------------ | --------------------------------------------- |
| **Fail Fast**                  | Fehler früh erkennen und stoppen              |
| **Try-Catch verwenden**        | Aber nicht überall (nur wo sinnvoll!)         |
| **Eigene Exception-Klassen**   | Z. B. `InvalidUserException`, `DataError`     |
| **Loggen statt schlucken**     | Fehler nicht einfach ignorieren               |
| **Fehlertypen differenzieren** | Logikfehler ≠ Netzwerkfehler ≠ Benutzerfehler |

### 🧱 Beispiele: Eigene Fehlerklassen

#### 🔹 Java

```java
public class InvalidUserException extends RuntimeException {
    public InvalidUserException(String message) {
        super(message);
    }
}
```

#### 🔹 Python

```python
class InvalidUserError(Exception):
    def __init__(self, message):
        super().__init__(message)
```

#### 🔹 TypeScript

```ts
class InvalidUserError extends Error {
  constructor(message: string) {
    super(message)
    this.name = "InvalidUserError"
  }
}
```

---

## 🧾 Standards und Formate

| Bereich                   | Standard / Guideline                                                                             |
| ------------------------- | ------------------------------------------------------------------------------------------------ |
| **Logging (Format)**      | [RFC 5424](https://tools.ietf.org/html/rfc5424) (Syslog), JSON logging                           |
| **PHP**                   | [PSR-3: Logger Interface](https://www.php-fig.org/psr/psr-3/)                                    |
| **.NET Logging**          | [Microsoft Logging Extensions](https://learn.microsoft.com/en-us/dotnet/core/extensions/logging) |
| **Kubernetes Logs**       | JSON logs, stdout/stderr, EFK/ELK stack                                                          |
| **OpenTelemetry**         | Logging, Tracing & Metrics Standard                                                              |
| **Sentry, Datadog, etc.** | Logging/Monitoring SaaS-Plattformen                                                              |

---

## 🔄 Optional: Logging-Integrationen

* **ELK Stack** (Elasticsearch, Logstash, Kibana)
* **EFK Stack** (Fluentd statt Logstash)
* **Grafana Loki**
* **Sentry (Fehlerberichte + Logs)**
* **Datadog / New Relic**

---
