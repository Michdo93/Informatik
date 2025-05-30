# Konventionen & Styleguides

Ja, es gibt eine ganze Reihe etablierter **Konventionen und Styleguides** – meist pro **Programmiersprache**, manchmal auch **technologie- oder frameworkbezogen**. Diese Styleguides legen fest, **wie Code lesbar, konsistent und wartbar** geschrieben werden soll, inklusive Namenskonventionen, Einrückungen, Leerzeichen, Kommentaren und Dateistrukturen.

---

## 🏁 **Was ist ein Styleguide?**

Ein **Styleguide** oder **Code Conventions Guide** ist eine Sammlung von **Regeln und Empfehlungen** für:

* **Benennung von Variablen, Klassen, Methoden**
* **Einrückung und Zeilenumbrüche**
* **Kommentare und Dokumentation**
* **Dateistruktur und Organisation**
* **Fehlermeldungen, Logging, usw.**

---

## 🧭 Übersicht gängiger Styleguides nach Sprache

| Sprache        | Offizielle / Bekannte Styleguides                                                                            |
| -------------- | ------------------------------------------------------------------------------------------------------------ |
| **Java**       | Oracle Java Code Conventions, Google Java Style Guide                                                        |
| **Python**     | [PEP 8](https://peps.python.org/pep-0008/)                                                                   |
| **JavaScript** | [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript), Google JS Style, StandardJS           |
| **TypeScript** | [Microsoft TypeScript Guidelines](https://github.com/microsoft/TypeScript/wiki/Coding-guidelines), Airbnb TS |
| **C#**         | Microsoft C# Coding Conventions                                                                              |
| **C/C++**      | LLVM, Google C++ Style Guide                                                                                 |
| **Go**         | [Effective Go](https://golang.org/doc/effective_go.html), `gofmt`                                            |
| **Rust**       | [Rust Style Guide (rustfmt)](https://github.com/rust-lang/rustfmt)                                           |
| **PHP**        | [PSR-1, PSR-12 (PHP-FIG)](https://www.php-fig.org/)                                                          |
| **Swift**      | [Ray Wenderlich Swift Style Guide](https://github.com/raywenderlich/swift-style-guide), Apple Swift Guide    |
| **Kotlin**     | [JetBrains Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)                   |
| **HTML/CSS**   | Google HTML/CSS Guide, [Airbnb CSS Guide](https://github.com/airbnb/css)                                     |
| **Shell/Bash** | Google Shell Style Guide                                                                                     |

---

## 🔍 Beispiele für konkrete Regeln

### 🔹 **Python (PEP 8)**

* Snake case für Variablen: `user_name`
* 4 Leerzeichen pro Einrückung
* Zeilenlänge: 79 Zeichen
* Klassen: `PascalCase`
* Konstanten: `ALL_CAPS`

### 🔹 **JavaScript (Airbnb)**

* `camelCase` für Variablen & Funktionen
* `const` bevorzugen
* Keine `var`, nur `let`/`const`
* 2 Leerzeichen pro Einrückung
* Kein Semikolon-Verzicht (empfiehlt `;`!)

### 🔹 **Go**

* CamelCase für Exporte (`ExportedFunc`)
* 1 Importblock, sortiert
* Keine unnötigen Klammern
* `gofmt` als Standard für Codeformatierung

### 🔹 **PHP (PSR-12)**

* 4 Leerzeichen, kein Tab
* Klammern nach `function`, `if`, etc.
* Klassen und Methoden: PascalCase
* Methoden: camelCase
* Namespace und Autoloading (PSR-4-konform)

---

## 💼 Weitere nützliche Tools & Formate

| Tool / Standard     | Zweck                                    |
| ------------------- | ---------------------------------------- |
| **EditorConfig**    | Einheitliches Einrückungsverhalten       |
| **Prettier**        | Code-Formatter (JS, TS, HTML, CSS, etc.) |
| **ESLint / TSLint** | Linter für JavaScript / TypeScript       |
| **Black (Python)**  | Opinionated Code Formatter               |
| **rustfmt**         | Formatter für Rust                       |
| **gofmt**           | Formatter für Go                         |
| **php-cs-fixer**    | Style Fixer für PHP                      |

---

## 🧠 Bonus: Framework-spezifische Konventionen

| Framework           | Konventionen / Styleguides      |
| ------------------- | ------------------------------- |
| **React** (JS/TS)   | Airbnb + JSX Guidelines         |
| **Angular**         | Angular Style Guide (offiziell) |
| **Laravel (PHP)**   | PSR + Laravel-Spezifika         |
| **Django (Python)** | PEP 8 + Django Guidelines       |
| **Vue.js**          | Vue Style Guide (offiziell)     |
| **Spring (Java)**   | Java + Spring Best Practices    |
