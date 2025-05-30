
# 📟 JavaScript Styleguide (Beispiel)

## 1. **Allgemeines**

* Sprache: ES6+
* Style basiert auf [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
* UTF-8 Encoding

---

## 2. **Einrückung & Formatierung**

* 2 Leerzeichen pro Einrückung (kein Tab)
* Max. Zeilenlänge: 100–120 Zeichen
* Keine Leerzeichen vor Funktionsklammern (`foo()` nicht `foo ()`)
* Semikolons verpflichtend
* Zeilen mit Hängender Einrückung für lange Statements:

  ```js
  const result = someFunction(
    param1,
    param2,
    param3,
  );
  ```

---

## 3. **Variablennamen & Deklarationen**

* `camelCase` für Variablen und Funktionen
* `PascalCase` für Klassen und Komponenten
* Konstanten in `UPPER_SNAKE_CASE`
* Variablen mit `const` oder `let`, niemals `var`

---

## 4. **Kommentare**

* Einzeilige Kommentare mit `//`
* Mehrzeilige Kommentare mit `/** ... */` für Dokumentation (JSDoc)
* Beispiel JSDoc:

  ```js
  /**
   * Add two numbers.
   *
   * @param {number} a - First number
   * @param {number} b - Second number
   * @returns {number} Sum of a and b
   */
  function add(a, b) {
    return a + b;
  }
  ```

---

## 5. **Imports & Exports**

* Verwende ES6 Module Syntax `import` / `export`
* Sortiere Imports alphabetisch oder gruppiere nach Typ (Bibliotheken / intern)
* Beispiel:

  ```js
  import React from 'react';
  import { helper } from './utils';
  ```

---

## 6. **Fehlerbehandlung**

* Verwende `try/catch` nur bei wirklich nötig
* Nutze `Error`-Objekte mit aussagekräftigen Nachrichten
* Eigene Error-Klassen bei komplexeren Fehlern

---

## 7. **Logging**

* Verwende `console.log` nur für Debugging, in Produktion bessere Lösungen wie `winston`, `pino`
* Log-Level (sofern Framework): debug, info, warn, error

---

## 8. **Sonstiges**

* Keine globalen Variablen
* Vermeide komplexe verschachtelte Bedingungen (frühzeitig returnen)
* Schreibe kleine, gut testbare Funktionen
* Halte Komponenten/Module übersichtlich und thematisch fokussiert

---
