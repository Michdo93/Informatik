# Code-Formatierung

Dieser Ordner beschreibt, **wie Code aussehen soll**: Namen, Einrückung, Kommentare, Projektstruktur, Fehlermeldungen. Ziel ist Code, den auch andere (und man selbst in einem Jahr) ohne Mühe lesen können.

<!-- TOC -->
## Inhaltsverzeichnis

- [Allgemeine Themen](#allgemeine-themen)
- [Styleguides pro Sprache](#styleguides-pro-sprache)
- [Auf einen Blick: die wichtigsten Unterschiede](#auf-einen-blick-die-wichtigsten-unterschiede)
<!-- /TOC -->

## Allgemeine Themen

| Dokument | Inhalt |
| --- | --- |
| [Konventionen & Styleguides](Konventionen%20&%20Styleguides.md) | Was ein Styleguide ist und welche es pro Sprache gibt |
| [Namenskonventionen](Namenskonventionen.md) | camelCase, snake_case, PascalCase & Co. |
| [Einrückung & Zeilenumbrüche](Einrückung%20&%20Zeilenumbrüche.md) | Tabs vs. Spaces, Zeilenlänge, LF vs. CRLF, `.editorconfig` |
| [Kommentare & Dokumentation](Kommentare%20&%20Dokumentation.md) | Kommentarsyntax und Doku-Formate (Javadoc, Doxygen, Docstrings, …) |
| [Dateistruktur & Projektorganisation](Dateistruktur%20&%20Projektorganisation.md) | Typische Ordnerstrukturen je Sprache |
| [Fehlermeldungen, Logging & Exceptions](Fehlermeldungen,%20Logging%20&%20Exceptions.md) | Log-Level, strukturierte Logs, Fehlerklassen |
| [Heredoc](Heredoc.md) | Mehrzeilige Strings und Heredoc-Marker |

## Styleguides pro Sprache

| Sprache | Basis | Dokument |
| --- | --- | --- |
| Python | PEP 8 | [Beispiele/Python.md](Beispiele/Python.md) |
| JavaScript | Airbnb | [Beispiele/JavaScript.md](Beispiele/JavaScript.md) |
| C | Linux Kernel Coding Style | [Beispiele/C.md](Beispiele/C.md) |
| C++ | Google C++ Style Guide + C++ Core Guidelines | [Beispiele/C++.md](Beispiele/C++.md) |
| C# | Microsoft C# Coding Conventions | [Beispiele/CSharp.md](Beispiele/CSharp.md) |
| Java | Google Java Style Guide | [Beispiele/Java.md](Beispiele/Java.md) |

## Auf einen Blick: die wichtigsten Unterschiede

| | Python | JavaScript | C | C++ (Google) | C# | Java |
| --- | --- | --- | --- | --- | --- | --- |
| Einrückung | 4 Spaces | 2 Spaces | 4 Spaces / Tab | 2 Spaces | 4 Spaces | 2 Spaces |
| Klammerstil | – | K&R | K&R | K&R | Allman | K&R |
| Funktionen | `snake_case` | `camelCase` | `snake_case` | `PascalCase` | `PascalCase` | `camelCase` |
| Konstanten | `UPPER_CASE` | `UPPER_CASE` | `UPPER_CASE` | `kPascalCase` | `PascalCase` | `UPPER_CASE` |
| Private Felder | `_name` | `#name` | – | `name_` | `_name` | `name` |
| Interfaces | – | – | – | – | `ISensor` | `Sensor` |
| Doku-Format | Docstrings | JSDoc | Doxygen | Doxygen | XML-Doc | Javadoc |
| Formatter | Black / Ruff | Prettier | clang-format | clang-format | dotnet format | google-java-format |

> **Wichtigste Regel:** Ein bestehendes Projekt behält seinen Stil. Wer in fremdem Code arbeitet, passt sich an – auch wenn der eigene Lieblingsstil ein anderer ist.
