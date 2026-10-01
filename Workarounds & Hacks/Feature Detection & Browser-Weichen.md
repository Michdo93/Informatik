# 🔀 Feature Detection & Browser-Weichen

Wie stellt man sicher, dass Code in **verschiedenen Umgebungen** (Browser, Betriebssysteme, Python-Versionen, Gerätefirmware) funktioniert? Es gibt grob zwei Strategien – eine gute und eine schlechte.

<!-- TOC -->
## Inhaltsverzeichnis

- [User-Agent-Sniffing vs. Feature Detection](#user-agent-sniffing-vs-feature-detection)
- [Progressive Enhancement vs. Graceful Degradation](#progressive-enhancement-vs-graceful-degradation)
- [Vendor-Prefixe](#vendor-prefixe)
- [Feature Detection außerhalb des Browsers](#feature-detection-außerhalb-des-browsers)
- [Checkliste](#checkliste)
<!-- /TOC -->

## User-Agent-Sniffing vs. Feature Detection

| | User-Agent-Sniffing ❌ | Feature Detection ✅ |
| --- | --- | --- |
| Frage | „**Welcher** Browser bist du?“ | „**Kannst** du X?“ |
| Methode | `navigator.userAgent` auswerten | Prüfen, ob die Funktion existiert |
| Problem | User-Agents lügen, ändern sich, werden gefälscht; neue Browser unbekannt | Kaum – prüft genau das, was man braucht |

**Warum User-Agents lügen:** Fast jeder Browser beginnt mit `Mozilla/5.0`, Chrome gibt sich zusätzlich als Safari aus, Edge als Chrome … Das ist historisch gewachsen: Webseiten lieferten moderne Inhalte nur an bestimmte Browser aus, also gaben sich neue Browser als die alten aus. Ein schönes Beispiel dafür, wie ein Workaround den nächsten erzeugt.

```javascript
// ❌ Bad: guessing capabilities from the browser name
if (navigator.userAgent.includes("Chrome")) {
  useClipboardApi();
}

// ✅ Good: testing for the capability itself
if (navigator.clipboard && "writeText" in navigator.clipboard) {
  navigator.clipboard.writeText("192.168.10.21");
} else {
  fallbackCopyWithTextarea("192.168.10.21");
}
```

**In CSS** mit `@supports`:

```css
.dashboard {
  display: flex;                 /* fallback for old browsers */
}

@supports (display: grid) {
  .dashboard {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(12rem, 1fr));
  }
}
```

---

## Progressive Enhancement vs. Graceful Degradation

| Strategie | Vorgehen | Beispiel |
| --- | --- | --- |
| **Progressive Enhancement** | Mit einer **Basisversion** starten, die überall funktioniert, und für fähige Umgebungen **verbessern** | Dashboard funktioniert als einfaches HTML-Formular; mit JavaScript werden Live-Updates per WebSocket ergänzt |
| **Graceful Degradation** | Für moderne Umgebungen bauen und sicherstellen, dass ältere **sinnvoll abgestuft** funktionieren | Animierte Karte; ohne WebGL wird ein statisches Bild gezeigt |

Progressive Enhancement gilt als robuster, weil die Basis immer funktioniert.

---

## Vendor-Prefixe

Früher führten Browser neue CSS-Features mit **Herstellerpräfix** ein: `-webkit-` (Chrome/Safari), `-moz-` (Firefox), `-ms-` (IE/alter Edge), `-o-` (alter Opera).

```css
.box {
  -webkit-transition: opacity 0.3s;
  -moz-transition: opacity 0.3s;
  transition: opacity 0.3s;       /* standard property always last */
}
```

Heute: **Nicht von Hand schreiben**, sondern **Autoprefixer** (PostCSS) im Build einsetzen. Er ergänzt nur die Präfixe, die für die Zielbrowser (`browserslist`) nötig sind.

---

## Feature Detection außerhalb des Browsers

Das Prinzip „Fähigkeit prüfen statt Version raten“ gilt überall:

```python
import sys
import shutil

# Prefer checking capability ...
if hasattr(str, "removeprefix"):            # Python >= 3.9
    topic = "lab/temp".removeprefix("lab/")
else:
    topic = "lab/temp"[len("lab/"):]

# ... or check an explicit version if behaviour (not existence) differs
if sys.version_info < (3, 10):
    raise SystemExit("Python 3.10 or newer required")

# Check whether a command-line tool exists before using it
if shutil.which("sshpass") is None:
    print("sshpass not installed - run: sudo apt install sshpass")
```

```bash
# Shell: check for tools instead of assuming a distribution
if command -v apt-get >/dev/null 2>&1; then
    sudo apt-get install -y rsync
elif command -v dnf >/dev/null 2>&1; then
    sudo dnf install -y rsync
fi
```

```c
/* C: compile-time feature detection with the preprocessor */
#if defined(_WIN32)
    #include <windows.h>
    #define SLEEP_MS(ms) Sleep(ms)
#else
    #include <unistd.h>
    #define SLEEP_MS(ms) usleep((ms) * 1000)
#endif
```

In C/C++-Projekten übernimmt das Build-System die Prüfung (CMake `check_symbol_exists`, Autotools `./configure`) und erzeugt eine `config.h` mit `HAVE_...`-Makros (→ [Compiler & Build](../Compiler%20%26%20Build/README.md)).

---

## Checkliste

* [ ] Zielumgebungen **dokumentiert** (Browser, OS, Python-Version, Firmware).
* [ ] **Feature Detection** statt Namens- oder Versionsprüfung, wo möglich.
* [ ] **Fallback** für jedes optionale Feature.
* [ ] Präfixe und Polyfills **automatisiert** (Autoprefixer, Babel) statt von Hand.
* [ ] In den Zielumgebungen **wirklich testen** – auch auf dem alten Wandtablet im Labor.

---
