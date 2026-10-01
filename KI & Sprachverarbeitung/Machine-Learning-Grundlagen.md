# 🧠 Machine-Learning-Grundlagen

Viele Projektideen im Labor enthalten „KI“: Geräte im Kamerabild erkennen, Absichten in Sätzen verstehen, Gerätezustände vorhersagen. Hinter allen steckt **maschinelles Lernen (ML)**. Dieses Kapitel erklärt die Grundbegriffe, die man für solche Projekte braucht – und die typischen Fehler, die Ergebnisse wertlos machen.

<!-- TOC -->
## Inhaltsverzeichnis

- [Grundbegriffe](#grundbegriffe)
- [Arten des Lernens](#arten-des-lernens)
  - [Aufgabentypen](#aufgabentypen)
- [Daten aufteilen: Training, Validierung, Test](#daten-aufteilen-training-validierung-test)
  - [Overfitting und Underfitting](#overfitting-und-underfitting)
  - [Datenleck (Data Leakage)](#datenleck-data-leakage)
- [Qualität messen](#qualität-messen)
  - [Konfusionsmatrix](#konfusionsmatrix)
  - [Wahrscheinlichkeiten bewerten](#wahrscheinlichkeiten-bewerten)
  - [Immer mit einer Baseline vergleichen](#immer-mit-einer-baseline-vergleichen)
- [Beispiel: Zustandsvorhersage mit chronologischer Aufteilung](#beispiel-zustandsvorhersage-mit-chronologischer-aufteilung)
- [Transfer Learning und kleine Datensätze](#transfer-learning-und-kleine-datensätze)
- [Modelle auf Geräten ausführen](#modelle-auf-geräten-ausführen)
- [Modelle versionieren und betreiben](#modelle-versionieren-und-betreiben)
- [Checkliste für ML-Projekte](#checkliste-für-ml-projekte)
<!-- /TOC -->

## Grundbegriffe

| Begriff | Bedeutung | Beispiel |
| --- | --- | --- |
| **Modell** | Funktion mit lernbaren Parametern, die Eingaben auf Ausgaben abbildet | Klassifikator Satz → Intent |
| **Training** | Parameter so anpassen, dass das Modell die Beispieldaten gut abbildet | Logistische Regression auf Beispielsätzen |
| **Inferenz** | Das fertige Modell auf neue Daten anwenden | Neuer Satz → Intent |
| **Feature** (Merkmal) | Eingabegröße, aus der das Modell lernt | Uhrzeit, Wochentag, letzter Zustand |
| **Label** (Zielwert) | Die richtige Antwort im Trainingsbeispiel | `ON` / `OFF`, Intent-Name, Gerätename |
| **Datensatz** | Sammlung von Beispielen (Features + Labels) | 6 Monate Zustandsverlauf aus openHAB |
| **Hyperparameter** | Einstellungen, die **nicht** gelernt, sondern vorgegeben werden | Lernrate, Anzahl Bäume, Schwellwert |

---

## Arten des Lernens

| Art | Daten | Ziel | Beispiele im Smart Home |
| --- | --- | --- | --- |
| **Überwachtes Lernen** (*supervised*) | Beispiele **mit** Label | Label für neue Daten vorhersagen | Intent-Erkennung, Geräteerkennung im Bild, Zustandsvorhersage |
| **Unüberwachtes Lernen** (*unsupervised*) | Daten **ohne** Label | Struktur finden | Tagesroutinen clustern, Anomalien im Stromverbrauch erkennen |
| **Bestärkendes Lernen** (*reinforcement*) | Belohnung/Bestrafung für Aktionen | Strategie lernen | Heizungssteuerung, die Komfort und Energie abwägt |
| **Selbstüberwachtes Lernen** | Labels aus den Daten selbst | Repräsentationen lernen | Grundlage großer Sprach- und Bildmodelle |

> Häufige Verwechslung: Eine Anwendung, die **Feedback von Nutzenden** sammelt und gelegentlich neu trainiert, betreibt meist **überwachtes Lernen mit neuen Daten** (*Active Learning*, *Continual Learning*) – kein Reinforcement Learning im engeren Sinn.

### Aufgabentypen

| Typ | Ausgabe | Beispiel |
| --- | --- | --- |
| **Klassifikation** | Kategorie (+ Wahrscheinlichkeit) | Intent, Gerät im Bild, `ON`/`OFF` |
| **Regression** | Zahl | Raumtemperatur in einer Stunde |
| **Zeitreihenprognose** | Werteverlauf in der Zukunft | Stromverbrauch morgen |
| **Objekterkennung** | Position + Klasse im Bild | Wo im Kamerabild ist die Lampe? |

---

## Daten aufteilen: Training, Validierung, Test

Ein Modell muss auf **Daten bewertet werden, die es beim Training nicht gesehen hat** – sonst misst man nur, wie gut es auswendig gelernt hat.

| Teil | Anteil (typisch) | Zweck |
| --- | --- | --- |
| **Trainingsdaten** | 60–80 % | Parameter lernen |
| **Validierungsdaten** | 10–20 % | Hyperparameter und Schwellwerte einstellen, Modelle vergleichen |
| **Testdaten** | 10–20 % | **Einmal** am Ende: ehrliche Abschätzung der Qualität |

### Overfitting und Underfitting

| | Underfitting | Gute Passung | Overfitting |
| --- | --- | --- | --- |
| Training | schlecht | gut | **sehr** gut |
| Test | schlecht | gut | **schlecht** |
| Ursache | Modell zu einfach, schlechte Features | – | Modell zu komplex, zu wenige Daten – es lernt Zufälligkeiten auswendig |

### Datenleck (Data Leakage)

Ein **Datenleck** liegt vor, wenn Informationen aus den Testdaten ins Training gelangen. Das Modell wirkt dann gut, versagt aber im Einsatz. Typische Ursachen:

* **Zeitreihen zufällig aufgeteilt:** Bei Zustandsvorhersagen darf man **nicht** zufällig mischen. Sonst lernt das Modell aus „morgen“, um „heute“ vorherzusagen. Richtig: **chronologisch** trennen – z. B. Januar bis Mai trainieren, Juni testen.
* **Features, die die Antwort verraten:** Wer die Wahrscheinlichkeit „Licht an um 21 Uhr“ vorhersagt, darf nicht den Zustand **um 21 Uhr** als Feature verwenden.
* **Dieselben Fotos** eines Geräts in Training und Test – die Erkennung wirkt dann besser, als sie bei neuen Aufnahmen ist.

---

## Qualität messen

### Konfusionsmatrix

Für eine Ja/Nein-Vorhersage (z. B. „Licht wird um 21 Uhr an sein“):

| | **Tatsächlich an** | **Tatsächlich aus** |
| --- | --- | --- |
| **Vorhergesagt an** | Richtig positiv (TP) | Falsch positiv (FP) |
| **Vorhergesagt aus** | Falsch negativ (FN) | Richtig negativ (TN) |

| Kennzahl | Formel | Frage |
| --- | --- | --- |
| **Accuracy** | (TP + TN) / alle | Wie oft liegt das Modell richtig? |
| **Precision** | TP / (TP + FP) | Wenn es „an“ sagt – wie oft stimmt das? |
| **Recall** | TP / (TP + FN) | Von allen echten „an“ – wie viele findet es? |
| **F1-Score** | 2 · P · R / (P + R) | Kompromiss aus Precision und Recall |

> **Vorsicht bei unausgewogenen Daten:** Ist das Licht 95 % der Zeit aus, erreicht ein Modell, das **immer** „aus“ sagt, 95 % Accuracy – und ist völlig nutzlos. Deshalb Precision/Recall betrachten und mit einer **Baseline** vergleichen.

### Wahrscheinlichkeiten bewerten

Gibt ein Modell **Wahrscheinlichkeiten** aus („94 % Kaffeemaschine an“), sollten diese **kalibriert** sein: Von allen Vorhersagen mit 90 % sollten tatsächlich etwa 90 % eintreten. Messgrößen dafür sind der **Brier-Score** und das **Log-Loss**; ein **Kalibrierungsdiagramm** zeigt Abweichungen anschaulich.

### Immer mit einer Baseline vergleichen

Eine **Baseline** ist das einfachste sinnvolle Verfahren. Ein komplexes Modell ist nur dann gut, wenn es die Baseline **deutlich** schlägt.

| Aufgabe | Mögliche Baseline |
| --- | --- |
| Zustandsvorhersage | Häufigkeit des Zustands zur selben Uhrzeit an denselben Wochentagen |
| Intent-Erkennung | Schlüsselwort-Regeln |
| Bilderkennung | Template Matching mit OpenCV |

---

## Beispiel: Zustandsvorhersage mit chronologischer Aufteilung

```python
# pip install scikit-learn pandas numpy
import numpy as np
import pandas as pd
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.metrics import brier_score_loss, roc_auc_score

# --- synthetic history: light in the living room, hourly for 26 weeks ---
rng = np.random.default_rng(42)
times = pd.date_range("2026-01-05", periods=26 * 7 * 24, freq="h")
df = pd.DataFrame({"time": times})
df["hour"] = df.time.dt.hour
df["weekday"] = df.time.dt.weekday
evening = df.hour.between(19, 22)
weekend_morning = (df.weekday >= 5) & df.hour.between(9, 11)
p_on = 0.05 + 0.8 * evening + 0.6 * weekend_morning
df["on"] = rng.random(len(df)) < p_on.clip(0, 0.95)

# --- features: cyclic encoding of time, so that 23:00 and 00:00 are close ---
df["hour_sin"] = np.sin(2 * np.pi * df.hour / 24)
df["hour_cos"] = np.cos(2 * np.pi * df.hour / 24)
df["is_weekend"] = (df.weekday >= 5).astype(int)
features = ["hour_sin", "hour_cos", "weekday", "is_weekend"]

# --- chronological split: never shuffle time series ---
split = df.time < "2026-06-01"
train, test = df[split], df[~split]

model = GradientBoostingClassifier().fit(train[features], train["on"])
p_model = model.predict_proba(test[features])[:, 1]

# --- baseline: frequency per (weekday, hour) in the training period ---
freq = train.groupby(["weekday", "hour"])["on"].mean()
p_base = test.set_index(["weekday", "hour"]).index.map(freq).to_numpy()

for name, p in [("baseline", p_base), ("model", p_model)]:
    print(f"{name:8}  AUC={roc_auc_score(test['on'], p):.3f}  Brier={brier_score_loss(test['on'], p):.3f}")

# --- question: probability that the light is on next Saturday at 10:00 / 21:00 ---
query = pd.DataFrame({"hour": [10, 21], "weekday": [5, 5]})
query["hour_sin"] = np.sin(2 * np.pi * query.hour / 24)
query["hour_cos"] = np.cos(2 * np.pi * query.hour / 24)
query["is_weekend"] = 1
for hour, p in zip(query.hour, model.predict_proba(query[features])[:, 1]):
    print(f"Saturday {hour:02d}:00 -> {p:.0%}")
```

Auf diesen synthetischen Daten ist die einfache Häufigkeits-Baseline praktisch genauso gut wie das Modell – weil das Verhalten nur von Uhrzeit und Wochentag abhängt. Ein Modell lohnt sich erst, wenn **weitere Features** (Anwesenheit, Wetter, Kalender, Zustände anderer Geräte) echte Zusatzinformation liefern. Genau diese Erkenntnis ist ein wertvolles Ergebnis.

---

## Transfer Learning und kleine Datensätze

Ein Bildklassifikator von Grund auf braucht Zehntausende Bilder. Für „erkenne **unsere** fünf Lampen“ gibt es aber nur eine Handvoll Fotos. Die Lösung ist **Transfer Learning**:

1. Ein **vortrainiertes Modell** (z. B. MobileNet, EfficientNet) hat auf Millionen Bildern gelernt, allgemeine Merkmale (Kanten, Formen, Texturen) zu erkennen.
2. Man **friert** diese Schichten ein und trainiert nur die **letzte Schicht** (den Klassifikator) mit den eigenen Bildern neu.
3. Schon wenige Dutzend Bilder pro Klasse können dann genügen.

Alternative ohne Training: Bilder mit dem vortrainierten Modell in **Merkmalsvektoren (Embeddings)** umwandeln und ein neues Bild der Klasse mit dem **ähnlichsten** gespeicherten Vektor zuordnen (*Nearest Neighbour*). Neue Geräte lassen sich dann hinzufügen, indem man einfach ihre Embeddings speichert – ganz ohne neues Training.

---

## Modelle auf Geräten ausführen

| Begriff | Bedeutung |
| --- | --- |
| **Edge / On-Device AI** | Inferenz direkt auf Smartphone, Raspberry Pi oder Mikrocontroller statt in der Cloud |
| **On-Device Training** | Das Gerät passt das Modell selbst an (meist nur die letzte Schicht) |
| **Quantisierung** | Parameter mit geringerer Genauigkeit speichern (z. B. 8 Bit statt 32 Bit) – kleiner, schneller, minimal ungenauer |
| **Austauschformate** | **ONNX**, **TensorFlow Lite** (inzwischen *LiteRT*), Core ML – Modelle einmal trainieren, auf verschiedenen Geräten ausführen |
| **Beschleuniger** | GPU, NPU, TPU (z. B. Coral), Hailo-Module für den Raspberry Pi |

> Frameworks und APIs in diesem Bereich ändern sich schnell (Umbenennungen, abgekündigte Bibliotheken). Vor Projektbeginn prüfen, ob eine Bibliothek noch gepflegt wird.

---

## Modelle versionieren und betreiben

* **Daten, Code und Modell gehören zusammen:** Zu jedem Modell festhalten, mit welchen Daten, welchem Code und welchen Hyperparametern es trainiert wurde.
* **Versionsnummer und Zeitstempel** für jedes trainierte Modell; ältere Versionen aufbewahren, um bei Verschlechterung zurückzuwechseln.
* **Messwerte mitspeichern:** Testergebnisse jeder Version, damit Vergleiche möglich sind.
* **Drift beobachten:** Ändert sich das Verhalten (neue Geräte, neue Bewohner, Jahreszeit), verschlechtert sich ein Modell schleichend.
* **Werkzeuge:** Für größere Projekte z. B. MLflow oder DVC; für kleinere reicht eine saubere Ordnerstruktur mit Metadaten-Datei pro Modell.

---

## Checkliste für ML-Projekte

* [ ] Problem als Aufgabentyp formuliert (Klassifikation, Regression, …)
* [ ] **Baseline** definiert und gemessen
* [ ] Daten **chronologisch** bzw. ohne Überschneidung aufgeteilt
* [ ] Passende **Metrik** gewählt (nicht nur Accuracy)
* [ ] Ergebnisse auf **Testdaten** berichtet, die nie für Entscheidungen genutzt wurden
* [ ] Modelle, Daten und Code versioniert
* [ ] Datenschutz geklärt: Welche personenbezogenen Daten (Anwesenheit, Gewohnheiten, Sprache, Bilder) werden verarbeitet und gespeichert?

---
