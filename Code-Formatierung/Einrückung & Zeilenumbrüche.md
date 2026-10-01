# Einrückung & Zeilenumbrüche

Auch für **Einrückung (Indentation)** und **Zeilenumbrüche (Line Breaks / Line Endings)** gibt es in fast allen Programmiersprachen **Styleguides oder Konventionen**, die oft sogar **technisch durchgesetzt** werden (z. B. durch Linter oder Formatter).

---

<!-- TOC -->
## Inhaltsverzeichnis

- [🧭 Übersicht: Standards für Einrückung & Zeilenumbrüche](#-übersicht-standards-für-einrückung--zeilenumbrüche)
  - [🔹 Einrückung](#-einrückung)
  - [🔹 Zeilenumbrüche (Line Breaks)](#-zeilenumbrüche-line-breaks)
- [📚 Sprachempfehlungen im Detail](#-sprachempfehlungen-im-detail)
- [🔧 Tools zur Durchsetzung](#-tools-zur-durchsetzung)
- [📄 Beispiel: .editorconfig](#-beispiel-editorconfig)
- [🧠 Sonstige Formatierungsrichtlinien (ergänzend)](#-sonstige-formatierungsrichtlinien-ergänzend)
<!-- /TOC -->

## 🧭 Übersicht: Standards für Einrückung & Zeilenumbrüche

### 🔹 Einrückung

| Regel                                  | Beschreibung                                                       |
| -------------------------------------- | ------------------------------------------------------------------ |
| **Größe:** 2 oder 4 Leerzeichen        | Je nach Sprache/Team — Tabs sind selten empfohlen                  |
| **Tabs vs. Spaces:** meist Spaces      | Tabs führen zu uneinheitlicher Darstellung, deshalb meist `Spaces` |
| **Blockeinrückung:**                   | Bei Codeblöcken (if, for, class, etc.) wird eingerückt             |
| **Klammerstil beeinflusst Einrückung** | z. B. Allman vs. K\&R Stil in C-artigen Sprachen                   |

### 🔹 Zeilenumbrüche (Line Breaks)

| Regel                        | Beschreibung                                               |
| ---------------------------- | ---------------------------------------------------------- |
| **Unix (LF, `\n`) Standard** | Windows: CRLF (`\r\n`), Unix/Linux/macOS: LF (`\n`)        |
| **Maximale Zeilenlänge**     | PEP 8: 79 Zeichen, Google: 100, viele Linter: 120 oder 140 |
| **Logische Zeilenumbrüche**  | Lange Ausdrücke/Methodenaufrufe werden lesbar umgebrochen  |
| **Trailing commas**          | Oft erlaubt/empfohlen für einfachere Diffbarkeit           |

---

## 📚 Sprachempfehlungen im Detail

| Sprache        | Einrückung                         | Zeilenlänge / Line Break    |
| -------------- | ---------------------------------- | --------------------------- |
| **Python**     | 4 Spaces (PEP 8)                   | 79 Zeichen empfohlen, `\n`  |
| **JavaScript** | 2 Spaces (Airbnb), 4 optional      | 100–120 Zeichen             |
| **Java**       | 4 Spaces (Google, Oracle)          | 100 Zeichen                 |
| **C#**         | 4 Spaces (Microsoft)               | 120 Zeichen                 |
| **Go**         | Tabs! (von `gofmt`)                | 100 Zeichen, `\n`           |
| **Rust**       | 4 Spaces (`rustfmt`)               | 100 Zeichen, `\n`           |
| **PHP**        | 4 Spaces (PSR-12)                  | max. 120, empfohlen: 80–100 |
| **C/C++**      | 2 oder 4 Spaces (Google C++ Guide) | 80–100 Zeichen              |
| **TypeScript** | 2 Spaces (Airbnb, TSLint)          | 100–120 Zeichen             |
| **Swift**      | 4 Spaces (Apple / RayWenderlich)   | 120 Zeichen                 |

---

## 🔧 Tools zur Durchsetzung

| Tool            | Sprachen         | Zweck                                 |
| --------------- | ---------------- | ------------------------------------- |
| `EditorConfig`  | Alle             | Einheitliche Editor-Einstellungen     |
| `Prettier`      | JS/TS/HTML/CSS   | Automatischer Formatter (2/4 Spaces)  |
| `Black`         | Python           | Opinionated Formatter (PEP 8-konform) |
| `gofmt`         | Go               | Erzwingt Tabs + Standardformat        |
| `rustfmt`       | Rust             | Erzwingt 4-Spaces, Styleguide-konform |
| `php-cs-fixer`  | PHP (PSR-12)     | Style-Enforcement                     |
| `clang-format`  | C/C++/Java/Obj-C | Einheitliche Formatierung             |
| `dotnet-format` | C# (.NET)        | Formatierung nach MS-Standards        |

---

## 📄 Beispiel: `.editorconfig`

```ini
# .editorconfig
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.js]
indent_size = 2
```

---

## 🧠 Sonstige Formatierungsrichtlinien (ergänzend)

* Leerzeile nach Imports
* Methoden voneinander durch Leerzeile trennen
* Kein Leerzeichen vor Klammer bei Funktionsaufrufen (`foo()` statt `foo ()`)
* Einrücken in Klammern:

  ```javascript
  if (true) {
    doSomething();
  }
  ```

---
