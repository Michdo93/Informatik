# 🧱 Polyfill & Shim

Ein **Polyfill** ist Code, der eine **fehlende Standardfunktion** in einer älteren Laufzeitumgebung (meist einem Browser) **nachrüstet**, sodass moderner Code dort trotzdem funktioniert.

<!-- TOC -->
## Inhaltsverzeichnis

- [Woher kommt der Name?](#woher-kommt-der-name)
- [Wie funktioniert ein Polyfill?](#wie-funktioniert-ein-polyfill)
- [Polyfill, Shim, Ponyfill, Transpiler](#polyfill-shim-ponyfill-transpiler)
- [Polyfills außerhalb des Browsers](#polyfills-außerhalb-des-browsers)
- [Sicherheit: Der polyfill.io-Vorfall (2024)](#sicherheit-der-polyfillio-vorfall-2024)
- [Brauche ich heute noch Polyfills?](#brauche-ich-heute-noch-polyfills)
<!-- /TOC -->

## Woher kommt der Name?

Die Vermutung liegt nahe: **Ja, die Analogie zum Bohrloch-Stopfen stimmt – mit einer kleinen Präzisierung.**

* **Polyfilla** ist eine in **Großbritannien** sehr bekannte Marke für **Spachtelmasse** (vergleichbar mit „Moltofill“ in Deutschland). Damit stopft man **Löcher und Risse in Wänden** – also genau das, was man nach einem falsch gebohrten Loch macht.
* Der britische Webentwickler **Remy Sharp** prägte den Begriff *Polyfill* um **2009/2010** (er beschrieb es im Blogartikel *„What is a Polyfill?“*, 2010). Er suchte ein Wort für Code, der **„die Löcher im Browser zuspachtelt“** – also die Lücken zwischen dem, was der Standard verspricht, und dem, was ein alter Browser kann.
* Er wollte bewusst nicht „Shim“ sagen, weil das damals für viele verschiedene Dinge verwendet wurde. Das „poly“ passt außerdem zu „viele“ – ein Polyfill soll in **vielen** Browsern gleich funktionieren.

> **Merksatz:** Ein Polyfill spachtelt die Löcher in der Wand (des Browsers) zu, damit die Tapete (dein Code) glatt aufliegt. Nach der Renovierung (Browser-Update) ist die Spachtelmasse unsichtbar – und sollte idealerweise gar nicht mehr nötig sein.

---

## Wie funktioniert ein Polyfill?

Grundprinzip: **Prüfen, ob die Funktion fehlt – nur dann nachrüsten.**

```javascript
// Polyfill for Array.prototype.includes (ES2016) - for very old browsers
if (!Array.prototype.includes) {
  Object.defineProperty(Array.prototype, "includes", {
    value: function (search, fromIndex) {
      const arr = Object(this);
      const len = arr.length >>> 0;
      let i = Math.max(fromIndex | 0, 0);
      for (; i < len; i++) {
        if (arr[i] === search || (Number.isNaN(arr[i]) && Number.isNaN(search))) {
          return true;
        }
      }
      return false;
    },
    configurable: true,
    writable: true,
  });
}

console.log([1, 2, NaN].includes(NaN)); // true - native or polyfilled
```

Wichtig:

* Ein Polyfill implementiert **exakt das Standardverhalten** (hier inkl. `NaN`-Sonderfall), nicht „ungefähr“.
* Er wird nur aktiv, wenn die native Funktion fehlt – native Implementierungen sind schneller.
* Polyfills verändern **globale Objekte** (Prototypen) – eine Form von [Monkey Patching](Monkey%20Patching.md), aber nach Standard.

---

## Polyfill, Shim, Ponyfill, Transpiler

| Begriff | Bedeutung | Beispiel |
| --- | --- | --- |
| **Polyfill** | Rüstet eine **Standard-API** nach, sodass der Code sie nutzen kann, als wäre sie nativ | `Promise`, `fetch`, `Array.prototype.includes` |
| **Shim** | Allgemeiner: eine **Zwischenschicht**, die eine API bereitstellt oder abfängt – nicht zwingend standardkonform | `es5-shim`, `html5shiv` (alte IE-Versionen) |
| **Ponyfill** | Wie Polyfill, **verändert aber keine globalen Objekte**, sondern wird importiert | `import includes from "array-includes"` |
| **Transpiler** | Übersetzt neue **Syntax** in alte (Syntax kann man nicht polyfillen!) | Babel, TypeScript: `a?.b` → `a == null ? void 0 : a.b` |
| **Fallback** | Alternative, wenn etwas nicht verfügbar ist | Videoformat WebM → MP4 |

> Der Begriff **Shim** kommt aus dem Handwerk: Ein *shim* ist eine dünne **Unterlegscheibe oder ein Keil**, mit dem man Spalten ausgleicht – etwa unter einem wackelnden Tischbein. Auch hier also: Unebenheiten ausgleichen.

**Syntax vs. API:**

```javascript
// API (can be polyfilled): a function/object that may be missing
Array.prototype.at, Object.hasOwn, structuredClone, fetch

// Syntax (needs a transpiler): the parser of an old browser does not understand it
const x = obj?.prop ?? "default";   // optional chaining, nullish coalescing
class A { #private = 1; }           // private fields
```

---

## Polyfills außerhalb des Browsers

Das Prinzip gibt es überall, wo Laufzeitumgebungen unterschiedlich alt sind:

| Umgebung | Beispiel |
| --- | --- |
| **Python** | `typing_extensions` bringt neue Typing-Features in alte Python-Versionen; `importlib_metadata` / `importlib_resources` als Backports |
| **Python** | `try: import tomllib  except ImportError: import tomli as tomllib` (TOML-Parser erst ab 3.11 eingebaut) |
| **C/C++** | Eigene Implementierung von `strlcpy`, wenn die libc sie nicht hat (`#ifndef HAVE_STRLCPY`) |
| **Java** | Backport-Bibliotheken (z. B. ThreeTen-Backport für `java.time` auf Java 6/7) |
| **Node.js** | `node-fetch` vor Node 18, `core-js` |

```python
# "Polyfill" in Python: use the stdlib module if available, else the backport
try:
    import tomllib                      # Python >= 3.11
except ModuleNotFoundError:             # Python <= 3.10
    import tomli as tomllib             # pip install tomli

with open("pyproject.toml", "rb") as f:
    config = tomllib.load(f)
```

---

## Sicherheit: Der polyfill.io-Vorfall (2024)

Viele Websites luden Polyfills über den Dienst **polyfill.io** per `<script src="https://cdn.polyfill.io/...">`. Der Dienst lieferte je nach Browser automatisch die passenden Polyfills aus.

* Anfang **2024** wurden Domain und GitHub-Projekt an eine neue Firma verkauft.
* Im **Juni 2024** wurde bekannt, dass über die Domain **Schadcode** an Besucher (vor allem mobile Nutzer) ausgeliefert wurde – Weiterleitungen auf betrügerische Seiten. Betroffen waren Schätzungen zufolge **über 100 000 Websites**.
* Cloudflare, Fastly und andere stellten **sichere Spiegel** bereit; die Domain wurde später abgeschaltet.

Das ist ein klassischer **Supply-Chain-Angriff**: Man vertraut fremdem Code, der sich jederzeit ändern kann.

**Lehren daraus:**

* [ ] Externe Skripte möglichst **selbst hosten** oder über den Build einbinden (npm, `core-js`), statt sie live von fremden CDNs zu laden.
* [ ] Wenn CDN, dann mit **Subresource Integrity (SRI)**: `<script src="…" integrity="sha384-…" crossorigin="anonymous">`. Dann lädt der Browser nur exakt die erwartete Datei.
* [ ] **Content Security Policy (CSP)** setzen, die nur erlaubte Quellen zulässt (eine Form von [Whitelisting](../Zugriffskontrolle/Whitelist%20%26%20Blacklist.md)).
* [ ] Regelmäßig prüfen, ob Polyfills überhaupt noch nötig sind – die meisten sind heute überflüssig.

---

## Brauche ich heute noch Polyfills?

Seit dem Ende des Internet Explorer (Support-Ende 2022) und der automatischen Updates aller gängigen Browser **deutlich seltener**. Vorgehen:

1. **Zielgruppe definieren:** Welche Browser müssen unterstützt werden? (z. B. per `browserslist`: `> 0.5%, last 2 versions, not dead`)
2. Auf **[caniuse.com](https://caniuse.com/)** prüfen, ob ein Feature dort unterstützt wird.
3. Nur das nötigste nachrüsten, automatisiert über Babel + `core-js` mit `useBuiltIns: "usage"`.
4. Bei Embedded-Browsern (alte Kiosk-Systeme, Smart-TVs, Wandtablets im Labor) genau hinsehen – dort laufen oft **sehr alte** WebViews.

---
