# Kommentare

Kommentare sind in jeder Programmiersprache ein wichtiger Bestandteil, um Code zu dokumentieren und lesbarer zu machen.

## Übersicht einzeilige & mehrzeilige Kommentare

Hier ist eine Übersicht, wie **einfache (einzeilige)** und **mehrzeilige Kommentare** in verschiedenen Programmiersprachen funktionieren:

---

### 🔹 **C / C++ / Java / JavaScript / C#**

* **Einzeilig:** `// Kommentar`
* **Mehrzeilig:**

  ```c
  /* 
     Mehrzeiliger
     Kommentar 
  */
  ```

---

### 🔹 **Python**

* **Einzeilig:** `# Kommentar`
* **Mehrzeilig:** (offiziell nur mehrere `#`, aber oft verwendet:)

  ```python
  """
  Mehrzeiliger
  Kommentar (eigentlich ein String, wird aber oft als Kommentar verwendet)
  """
  ```

---

### 🔹 **Ruby**

* **Einzeilig:** `# Kommentar`
* **Mehrzeilig:**

  ```ruby
  =begin
  Mehrzeiliger
  Kommentar
  =end
  ```

---

### 🔹 **HTML**

* **Kommentare:**

  ```html
  <!-- Kommentar -->
  ```

In HTML wird nicht unterschieden, ob ein Kommentar einzeilig oder mehrzeilig sein muss. Das liegt auch an Tags allgemein. Ich kann Tags bspw. wie folgt verwenden:

  ```html
  <div>Container</div>
  <div>
      Container
  </div>
  ```

Entsprechend kann ich dies auch für merhzeilige Kommentare anwenden:

  ```html
  <!--
      Kommentar
  -->
  ```

---

### 🔹 **CSS**

* **Einzeilig & Mehrzeilig:**

  ```css
  /* Kommentar */
  ```

---

### 🔹 **Shell (Bash)**

* **Einzeilig:** `# Kommentar`
* **Mehrzeilig:** Kein echtes mehrzeiliges Kommentar-Syntax, aber mehrere `#`-Zeilen werden verwendet.

---

### 🔹 **PHP**

* **Einzeilig:**

  ```php
  // Kommentar
  # Kommentar
  ```
* **Mehrzeilig:**

  ```php
  /*
    Mehrzeiliger Kommentar
  */
  ```

---

### 🔹 **Go**

* **Einzeilig:** `// Kommentar`
* **Mehrzeilig:**

  ```go
  /*
     Mehrzeiliger Kommentar
  */
  ```

---

### 🔹 **Swift**

* **Einzeilig:** `// Kommentar`
* **Mehrzeilig:**

  ```swift
  /*
     Mehrzeiliger Kommentar
  */
  ```

---

### 🔹 **Rust**

* **Einzeilig:** `// Kommentar`
* **Mehrzeilig:**

  ```rust
  /*
    Mehrzeiliger Kommentar
  */
  ```

---

## Dokumentationsstandards & -formate

Viele Programmiersprachen verwenden spezielle **Dokumentationsstandards** oder **-formate**, um Funktionen, Klassen, Parameter und Rückgabewerte zu beschreiben. Diese helfen nicht nur beim Lesen, sondern auch Tools wie IDEs oder Doku-Generatoren (z. B. Doxygen, Javadoc, phpDocumentor) verstehen und verarbeiten diese Kommentare.

Hier eine Übersicht der **gängigen Dokumentationsformate**, **zugehörige Sprachen** und jeweils ein **Beispiel**:

---

### 🟨 **1. Javadoc (für Java)**

* **Verwendet in:** Java, teilweise auch für Android-Entwicklung
* **Tools:** Javadoc (generiert HTML-Dokumentation)

**Beispiel:**

```java
/**
 * Berechnet die Summe zweier Zahlen.
 *
 * @param a Erste Zahl
 * @param b Zweite Zahl
 * @return Die Summe von a und b
 */
public int add(int a, int b) {
    return a + b;
}
```

---

### 🟩 **2. PHPDoc (für PHP)**

* **Verwendet in:** PHP
* **Tools:** phpDocumentor, IDEs wie PhpStorm

**Beispiel:**

```php
/**
 * Führt eine Division durch.
 *
 * @param float $a Dividend
 * @param float $b Divisor
 * @return float Ergebnis der Division
 * @throws Exception Wenn Division durch 0 erfolgt
 */
function divide(float $a, float $b): float {
    if ($b === 0.0) {
        throw new Exception("Division durch null");
    }
    return $a / $b;
}
```

---

### 🟦 **3. Doxygen (für C, C++, Java, Python, etc.)**

* **Verwendet in:** C, C++, Java, Objective-C, Python, usw.
* **Tools:** [Doxygen](https://www.doxygen.nl)

**Beispiel (C++):**

```cpp
/**
 * @brief Multipliziert zwei Zahlen.
 * 
 * @param x Erste Zahl
 * @param y Zweite Zahl
 * @return Produkt von x und y
 */
int multiply(int x, int y);
```

---

### 🟪 **4. Docstrings (für Python, PEP 257 Standard)**

* **Verwendet in:** Python
* **Tools:** Sphinx, IDEs

#### 🔸 Google-Style:

```python
def add(a: int, b: int) -> int:
    """
    Add two numbers.

    Args:
        a (int): First number.
        b (int): Second number.

    Returns:
        int: Sum of a and b.
    """
    return a + b
```

#### 🔸 NumPy-Style:

```python
def subtract(a, b):
    """
    Subtract two numbers.

    Parameters
    ----------
    a : int
        First number.
    b : int
        Second number.

    Returns
    -------
    int
        Difference of a and b.
    """
    return a - b
```

---

### 🟥 **5. XML Documentation Comments (für C#)**

* **Verwendet in:** C#
* **Tools:** Visual Studio, Sandcastle

**Beispiel:**

```csharp
/// <summary>
/// Addiert zwei Zahlen.
/// </summary>
/// <param name="a">Die erste Zahl</param>
/// <param name="b">Die zweite Zahl</param>
/// <returns>Die Summe der beiden Zahlen</returns>
public int Add(int a, int b) {
    return a + b;
}
```

---

### 🟫 **6. Rustdoc (für Rust)**

* **Verwendet in:** Rust
* **Tools:** `cargo doc`

**Beispiel:**

````rust
/// Addiert zwei Zahlen.
///
/// # Beispiele
///
/// ```
/// let result = my_crate::add(2, 3);
/// assert_eq!(result, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
````

---

### 🟧 **7. JsDoc (für JavaScript/TypeScript)**

* **Verwendet in:** JavaScript, TypeScript
* **Tools:** JsDoc, TypeScript-Compiler

**Beispiel:**

```js
/**
 * Gibt den Namen mit Begrüßung zurück.
 *
 * @param {string} name - Der Name des Benutzers.
 * @returns {string} Begrüßungstext.
 */
function greet(name) {
    return `Hallo, ${name}!`;
}
```

---

### Übersichtstabelle

| Dokumentationsstil | Programmiersprachen        | Tooling                   |
| ------------------ | -------------------------- | ------------------------- |
| Javadoc            | Java                       | `javadoc`                 |
| PHPDoc             | PHP                        | `phpDocumentor`, IDEs     |
| Doxygen            | C, C++, Java, Python, usw. | `doxygen`                 |
| Python Docstrings  | Python                     | `Sphinx`, `pdoc`, IDEs    |
| XML Docs (C#)      | C#                         | Visual Studio, Sandcastle |
| Rustdoc            | Rust                       | `cargo doc`               |
| JsDoc              | JavaScript, TypeScript     | `jsdoc`, IDEs             |

---
