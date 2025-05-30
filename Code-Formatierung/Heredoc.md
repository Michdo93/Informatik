# 📄 Heredoc-Beispiele in verschiedenen Programmiersprachen

Hier ist eine übersichtliche **Markdown-Tabelle mit Heredoc-Beispielen** für verschiedene Programmiersprachen, inklusive der gebräuchlichen Marker wie `EOF`, `EOT`, `DOC`, etc.

---

## ✅ Übersicht in Markdown

````markdown
| Sprache     | Marker-Beispiel | Heredoc-Beispiel |
|-------------|------------------|------------------|
| **PHP**     | `<<<EOF`         | ```php<br>$text = <<<EOF<br>Dies ist ein<br>mehrzeiliger Text.<br>EOF;<br>``` |
| **PHP (Nowdoc)** | `<<<'EOF'`     | ```php<br>$text = <<<'EOF'<br>Dies ist ein<br>mehrzeiliger Text.<br>EOF;<br>``` |
| **Perl**    | `<<EOF`          | ```perl<br>my $text = <<EOF;<br>Mehrzeilig<br>Text<br>EOF<br>``` |
| **Bash**    | `<<EOF`          | ```bash<br>cat <<EOF<br>Mehrzeilig<br>Text<br>EOF<br>``` |
| **Ruby**    | `<<EOF`          | ```ruby<br>text = <<EOF<br>Mehrzeilig<br>Text<br>EOF<br>``` |
| **Python**  | _kein Heredoc_   | ```python<br>text = """<br>Mehrzeilig<br>Text<br>"""<br>``` |
| **JavaScript** | _kein Heredoc_ | ```js<br>const text = `<br>Mehrzeilig<br>Text<br>`;<br>``` |
| **Haskell** | _kein Heredoc_   | ```haskell<br>text = "Mehrzeilig\\nText"<br>``` |
````

---

## 🔍 Codebeispiele ausführlich

### 🔸 PHP (Heredoc)

```php
$text = <<<EOT
Das ist ein
mehrzeiliger Text.
EOT;
```

### 🔸 PHP (Nowdoc – ohne Variablenauswertung)

```php
$text = <<<'DOC'
Das ist ein
mehrzeiliger Text ohne Variablen-Ersatz.
DOC;
```

### 🔸 Perl

```perl
my $text = <<EOF;
Dies ist ein
mehrzeiliger Text.
EOF
```

### 🔸 Bash / Shell

```bash
cat <<EOF
Mehrzeiliger
Text
EOF
```

### 🔸 Ruby

```ruby
text = <<DOC
Mehrzeiliger
Text in Ruby
DOC
```

### 🔸 Python (kein Heredoc, aber mehrzeiliger String)

```python
text = """
Mehrzeiliger
Text in Python
"""
```

### 🔸 JavaScript (Template Literal)

```javascript
const text = `
Mehrzeiliger
Text in JavaScript
`;
```

---

## 📘 Bedeutung typischer Heredoc-Marker

Die Marker wie `EOF`, `EOT`, `DOC` usw. sind in **Heredocs rein symbolisch**, aber sie haben meist eine traditionelle oder semantische Bedeutung – sie stehen **nicht für Befehle oder Schlüsselwörter**, sondern sind **frei wählbare Begriffe**, die oft folgende Bedeutungen haben:

| Marker | Bedeutung                      | Langform                      | Typischer Kontext / Anmerkung                                    |
| ------ | ------------------------------ | ----------------------------- | ---------------------------------------------------------------- |
| `EOF`  | **End of File**                | "End of File"                 | Häufigster Marker, traditionell aus der Unix-Welt übernommen     |
| `EOT`  | **End of Text**                | "End of Text" (ASCII-Zeichen) | Etwas formeller als `EOF`, gelegentlich bei Doku/Textausgabe     |
| `DOC`  | **Document**                   | "Document"                    | Wird oft verwendet, um HTML-, XML- oder Textinhalte zu markieren |
| `TEXT` | **Text**                       | "Text"                        | Einfacher, sprechender Marker, v. a. bei Beispieltexten          |
| `DATA` | **Data**                       | "Data"                        | Gern verwendet für strukturierte Inhalte (z. B. JSON, SQL, etc.) |
| `SQL`  | **Structured Query Language**  | "SQL"                         | Wird oft benutzt, um SQL-Abfragen zu umschließen                 |
| `HTML` | **HyperText Markup Language**  | "HTML"                        | Häufig, wenn man HTML in einem String speichert                  |
| `XML`  | **eXtensible Markup Language** | "XML"                         | Analog zu HTML, zur Strukturierung von Daten                     |
| `MSG`  | **Message**                    | "Message"                     | Praktisch, um lange Log- oder Fehlermeldungen einzuschließen     |

---

### 🔹 Beispiel (frei wählbarer Marker)

```php
$sql = <<<SQL
SELECT * FROM users
WHERE active = 1
ORDER BY created_at DESC;
SQL;
```

⚠️ Wichtig: Der Marker (z. B. `SQL`) muss **am Anfang und am Ende exakt gleich** geschrieben sein (Groß-/Kleinschreibung beachten) und **am Anfang der Zeile** stehen.

---

### 🔧 Welche Marker sollte man wann verwenden?

| Inhalt / Zweck    | Empfohlener Marker            |
| ----------------- | ----------------------------- |
| Allgemeiner Text  | `EOF`, `EOT`, `TEXT`          |
| Dokumentation     | `DOC`, `COMMENT`              |
| HTML / XML / JSON | `HTML`, `XML`, `JSON`, `DATA` |
| SQL-Abfragen      | `SQL`                         |
| Fehlermeldungen   | `MSG`, `ERROR`                |

---
