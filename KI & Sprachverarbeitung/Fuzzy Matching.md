# 🔤 Fuzzy Matching

**Fuzzy Matching** (unscharfe Suche, *approximate string matching*) findet Zeichenketten, die einer gesuchten Zeichenkette **ähnlich**, aber nicht exakt gleich sind. Es hilft überall dort, wo Menschen Text eingeben oder Spracherkennung Text erzeugt: Tippfehler, Abkürzungen, Groß-/Kleinschreibung, falsch erkannte Wörter.

<!-- TOC -->
## Inhaltsverzeichnis

- [Wofür braucht man das?](#wofür-braucht-man-das)
- [Editierdistanz – die Grundidee](#editierdistanz--die-grundidee)
- [Die wichtigsten Verfahren](#die-wichtigsten-verfahren)
  - [Hamming-Distanz](#hamming-distanz)
  - [Levenshtein-Distanz](#levenshtein-distanz)
  - [Damerau-Levenshtein-Distanz](#damerau-levenshtein-distanz)
  - [Weitere Verfahren](#weitere-verfahren)
- [Implementierung in Python](#implementierung-in-python)
  - [Levenshtein selbst gebaut](#levenshtein-selbst-gebaut)
  - [Mit RapidFuzz](#mit-rapidfuzz)
- [Praxis: Item-Namen zuordnen](#praxis-item-namen-zuordnen)
- [Schwellwerte und Grenzen](#schwellwerte-und-grenzen)
- [Fuzzy Matching, Synonyme oder NLU?](#fuzzy-matching-synonyme-oder-nlu)
<!-- /TOC -->

## Wofür braucht man das?

| Situation | Eingabe | Gemeint |
| --- | --- | --- |
| Tippfehler im Chat | `Wohnzimer` | `Wohnzimmer` |
| Vertauschte Buchstaben | `Lihct` | `Licht` |
| Abkürzung | `Wz Licht` | `Wohnzimmer Licht` |
| Spracherkennung (STT) | „Schalte das Licht im **Flohr** an“ | `Flur` |
| Gerätenamen | `kueche lautsprecher` | `tKueche_Sonos_Lautsprecher` |
| Mediensuche | „Spiel Bohemian Rapsody“ | *Bohemian Rhapsody* |

Ein exakter Vergleich (`==`, `in`) scheitert in all diesen Fällen. Fuzzy Matching berechnet stattdessen einen **Abstand** oder eine **Ähnlichkeit** und akzeptiert Treffer oberhalb eines **Schwellwerts**.

---

## Editierdistanz – die Grundidee

Die meisten Verfahren zählen, wie viele **Bearbeitungsschritte** nötig sind, um eine Zeichenkette in die andere zu überführen. Jeder Schritt kostet 1.

| Operation | Beispiel | Erlaubt bei |
| --- | --- | --- |
| **Einfügen** | `Lich` → `Licht` | Levenshtein, Damerau-Levenshtein |
| **Löschen** | `Lichtt` → `Licht` | Levenshtein, Damerau-Levenshtein |
| **Ersetzen** | `Kicht` → `Licht` | Hamming, Levenshtein, Damerau-Levenshtein |
| **Vertauschen** benachbarter Zeichen | `Lihct` → `Licht` | nur Damerau-Levenshtein |

Aus der Distanz lässt sich eine **normalisierte Ähnlichkeit** zwischen 0 und 1 (bzw. 0–100 %) berechnen:

```text
Ähnlichkeit = 1 − Distanz / max(Länge A, Länge B)
```

Beispiel: `Wohnzimer` (9 Zeichen) → `Wohnzimmer` (10 Zeichen) braucht **1** Einfügung → Ähnlichkeit = 1 − 1/10 = **90 %**.

---

## Die wichtigsten Verfahren

### Hamming-Distanz

* Nur für Zeichenketten **gleicher Länge**.
* Zählt die **Positionen, an denen sich die Zeichen unterscheiden** (nur Ersetzungen).
* Ursprünglich aus der Codierungstheorie (Richard Hamming, 1950) – dort zählt man unterschiedliche **Bits** zweier Codewörter.

```text
N u m b e r
L u m b e r
↑
1 Unterschied  →  Hamming-Distanz = 1
```

> Auf Bit-Ebene ist die Hamming-Distanz die Anzahl unterschiedlicher Bits: `N` = `1001110`, `L` = `1001100` unterscheiden sich in **genau einem Bit**, die Distanz der beiden Zeichen ist also ebenfalls 1. Für Textvergleiche ist Hamming wegen der Längenbedingung kaum geeignet – ein fehlender Buchstabe verschiebt alle folgenden Positionen.

### Levenshtein-Distanz

* Der Klassiker (Wladimir Levenshtein, 1965).
* Minimale Anzahl von **Einfügungen, Löschungen und Ersetzungen**.
* Funktioniert mit unterschiedlichen Längen.

| Von | Nach | Schritte | Distanz |
| --- | --- | --- | --- |
| `Rechnungnummer` | `Rechnungsnummer` | `s` einfügen | 1 |
| `Rechnungsnumer` | `Rechnungsnummer` | `m` einfügen | 1 |
| `Kicht` | `Licht` | `K` → `L` | 1 |
| `Lihct` | `Licht` | `h`→`c`, `c`→`h` | **2** |

### Damerau-Levenshtein-Distanz

* Wie Levenshtein, zusätzlich zählt das **Vertauschen zweier benachbarter Zeichen** als **ein** Schritt.
* Passt gut zu echten Tippfehlern – Buchstabendreher sind sehr häufig.

| Von | Nach | Levenshtein | Damerau-Levenshtein |
| --- | --- | --- | --- |
| `Lihct` | `Licht` | 2 | **1** (`hc` ↔ `ch`) |
| `Rehcnun` | `Rechnung` | 3 | **2** (Dreher + `g` einfügen) |

> In der Praxis wird meist die vereinfachte Variante **Optimal String Alignment (OSA)** implementiert. Sie unterscheidet sich von der „echten“ Damerau-Levenshtein-Distanz nur in Sonderfällen, in denen ein bereits vertauschtes Teilstück nochmals bearbeitet wird.

### Weitere Verfahren

| Verfahren | Idee | Gut für |
| --- | --- | --- |
| **Jaro-Winkler** | Ähnlichkeit über gemeinsame Zeichen und Dreher, **Präfixe** werden höher gewichtet | Kurze Wörter, Namen |
| **Token Sort / Token Set Ratio** | Wörter sortieren bzw. als Menge vergleichen | Unterschiedliche Wortreihenfolge: „Licht Küche“ ↔ „Küche Licht“ |
| **Partial Ratio** | Beste Übereinstimmung eines **Teilstücks** | Gesuchter Begriff ist Teil eines langen Satzes |
| **n-Gramme** | Vergleich überlappender Zeichenfolgen (`Lic`, `ich`, `cht`) | Große Datenmengen, Suchindizes |
| **Phonetische Verfahren** | Gleiche **Aussprache** → gleicher Code (Soundex, für Deutsch: **Kölner Phonetik**) | Namen, Spracherkennungsfehler: `Meier`/`Mayer`/`Maier` |

---

## Implementierung in Python

### Levenshtein selbst gebaut

Das Verfahren nutzt **dynamische Programmierung**: Eine Tabelle speichert die Distanz aller Präfix-Paare; jede Zelle ergibt sich aus ihren drei Nachbarn.

```python
def levenshtein(a: str, b: str) -> int:
    """Minimum number of insertions, deletions and substitutions."""
    previous = list(range(len(b) + 1))
    for i, char_a in enumerate(a, start=1):
        current = [i]
        for j, char_b in enumerate(b, start=1):
            current.append(min(
                previous[j] + 1,                        # deletion
                current[j - 1] + 1,                     # insertion
                previous[j - 1] + (char_a != char_b),   # substitution
            ))
        previous = current
    return previous[-1]


def osa_distance(a: str, b: str) -> int:
    """Damerau-Levenshtein (optimal string alignment): also counts adjacent swaps."""
    d = [[0] * (len(b) + 1) for _ in range(len(a) + 1)]
    for i in range(len(a) + 1):
        d[i][0] = i
    for j in range(len(b) + 1):
        d[0][j] = j
    for i in range(1, len(a) + 1):
        for j in range(1, len(b) + 1):
            cost = a[i - 1] != b[j - 1]
            d[i][j] = min(d[i - 1][j] + 1, d[i][j - 1] + 1, d[i - 1][j - 1] + cost)
            if i > 1 and j > 1 and a[i - 1] == b[j - 2] and a[i - 2] == b[j - 1]:
                d[i][j] = min(d[i][j], d[i - 2][j - 2] + 1)    # transposition
    return d[-1][-1]


def hamming(a: str, b: str) -> int:
    if len(a) != len(b):
        raise ValueError("Hamming distance needs strings of equal length")
    return sum(x != y for x, y in zip(a, b))


def similarity(a: str, b: str) -> float:
    longest = max(len(a), len(b)) or 1
    return 1 - levenshtein(a, b) / longest


print(levenshtein("Lihct", "Licht"), osa_distance("Lihct", "Licht"))        # 2 1
print(levenshtein("Rehcnun", "Rechnung"), osa_distance("Rehcnun", "Rechnung"))  # 3 2
print(hamming("Number", "Lumber"))                                         # 1
print(f"{similarity('Wohnzimer', 'Wohnzimmer'):.0%}")                      # 90%
```

### Mit RapidFuzz

Für den echten Einsatz eine optimierte Bibliothek verwenden. **[RapidFuzz](https://github.com/rapidfuzz/RapidFuzz)** ist der schnelle, aktiv gepflegte Nachfolger von *fuzzywuzzy* (MIT-Lizenz, in C++ implementiert):

```python
# pip install rapidfuzz
from rapidfuzz import fuzz, process
from rapidfuzz.distance import Levenshtein

rooms = ["Wohnzimmer", "Küche", "Bad", "Konferenz", "Multimedia", "IoT"]

print(Levenshtein.distance("Wohnzimer", "Wohnzimmer"))            # 1
print(fuzz.ratio("Wohnzimer", "Wohnzimmer"))                       # ~94.7 (normalised by total length)
print(fuzz.token_sort_ratio("Licht Küche", "Küche Licht"))         # 100.0
print(fuzz.partial_ratio("küche", "schalte das licht in der küche an"))  # 100.0

# best match from a list, only if similarity >= 80
match = process.extractOne("Konferens", rooms, scorer=fuzz.ratio, score_cutoff=80)
print(match)                                                       # ('Konferenz', 88.9, 3)
```

> **Achtung:** `fuzz.ratio` normiert anders als die Formel oben (über die **Summe** beider Längen). Werte verschiedener Bibliotheken und Funktionen sind daher **nicht direkt vergleichbar** – Schwellwerte immer mit echten Beispielen kalibrieren.

---

## Praxis: Item-Namen zuordnen

Ein typischer Einsatz im Smart Home: Freitext (aus Chat oder Spracherkennung) soll einem **openHAB-Item** zugeordnet werden. Fuzzy Matching funktioniert deutlich besser auf **sprechenden Bezeichnungen** (Labels, Synonymen) als auf technischen Item-Namen.

```python
from rapidfuzz import fuzz, process

aliases = {
    "licht küche": "iKueche_Licht",
    "küchenlicht": "iKueche_Licht",
    "lautsprecher küche": "iKueche_Sonos_Lautsprecher",
    "rollladen konferenz": "iKonferenz_Rollladen",
    "beamer": "iMultimedia_Beamer_Power",
}


def find_item(text: str, threshold: float = 75) -> str | None:
    result = process.extractOne(text.lower(), aliases.keys(),
                                scorer=fuzz.token_set_ratio, score_cutoff=threshold)
    return aliases[result[0]] if result else None


print(find_item("Kuechenlicht"))            # iKueche_Licht
print(find_item("Rolladen im Konferenz"))   # iKonferenz_Rollladen
print(find_item("Kaffeemaschine"))          # None -> ask the user instead of guessing
```

**Vorverarbeitung** verbessert die Treffer deutlich:

* alles **kleinschreiben**,
* **Umlaute normalisieren** (`ü` → `ue`) – oder konsequent nicht,
* **Satzzeichen** und **Füllwörter** (`das`, `bitte`, `im`) entfernen,
* technische Präfixe (`i`, `t`, `g`) und Unterstriche in Item-Namen auflösen.

---

## Schwellwerte und Grenzen

| Problem | Folge | Gegenmittel |
| --- | --- | --- |
| Schwellwert zu niedrig | **Falsche Treffer** – das falsche Gerät wird geschaltet | Lieber nachfragen als raten; bei Aktionen höhere Schwelle als bei Abfragen |
| Schwellwert zu hoch | Tippfehler werden nicht mehr erkannt | Mit echten Eingaben testen und kalibrieren |
| Kurze Wörter | `Bad` ↔ `Bar` hat 67 % Ähnlichkeit | Kurze Begriffe exakt oder mit Kontext vergleichen |
| Gleich geschriebene, verschiedene Dinge | „Licht **an**“ vs. „Licht **aus**“ unterscheiden sich nur in einem Buchstaben! | Schlüsselwörter für **Aktionen** nie unscharf vergleichen |
| Bedeutung | `anschalten` ↔ `einschalten` sind sich als Zeichenkette kaum ähnlich | **Synonyme** oder ein [NLU-Modell](Intents%2C%20Entities%20%26%20Confidence.md) |

> **Faustregel:** Fuzzy Matching korrigiert **Schreibweise**, aber nicht **Bedeutung**. Für unterschiedliche Formulierungen derselben Absicht braucht man Synonymlisten oder Intent-Erkennung.

---

## Fuzzy Matching, Synonyme oder NLU?

| Ansatz | Erkennt | Erkennt nicht | Aufwand |
| --- | --- | --- | --- |
| **Exakter Vergleich** | Genau definierte Befehle | Jede Abweichung | minimal |
| **Fuzzy Matching** | Tippfehler, Dreher, kleine STT-Fehler | Umformulierungen | gering |
| **Synonymlisten** | Bekannte alternative Wörter | Unbekannte Formulierungen | mittel (Pflege!) |
| **NLU / Intent-Klassifikation** | Neue Formulierungen derselben Absicht | Völlig unbekannte Absichten | mittel bis hoch (Trainingsdaten) |
| **LLM** | Fast beliebige Formulierungen, Kontext | – (aber: Halluzinationen, Ressourcen) | hoch |

In der Praxis kombiniert man: **NLU** erkennt die Absicht (*Licht schalten*), **Fuzzy Matching** ordnet das genannte Gerät oder den Raum einer bekannten Liste zu. Siehe [Intents, Entities & Confidence](Intents%2C%20Entities%20%26%20Confidence.md).

---
