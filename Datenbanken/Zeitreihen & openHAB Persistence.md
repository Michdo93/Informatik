# 📈 Zeitreihen & openHAB Persistence

Ein Smart Home erzeugt ununterbrochen **Zeitreihen**: Temperaturen, Schaltzustände, Stromverbrauch, Anwesenheit. Wer diese Verläufe speichert, kann Diagramme zeichnen, Fehler nachvollziehen, Zustände nach einem Neustart wiederherstellen – und Muster für **Vorhersagen** nutzen. In openHAB heißt dieser Mechanismus **Persistence**.

<!-- TOC -->
## Inhaltsverzeichnis

- [Was ist eine Zeitreihe?](#was-ist-eine-zeitreihe)
- [Zeitreihendatenbanken](#zeitreihendatenbanken)
- [openHAB Persistence](#openhab-persistence)
  - [Wozu?](#wozu)
  - [Konzept](#konzept)
  - [Strategien](#strategien)
  - [Strategie für Auswertungen](#strategie-für-auswertungen)
- [Daten auslesen](#daten-auslesen)
  - [Über die REST API](#über-die-rest-api)
  - [In Python auswerten](#in-python-auswerten)
- [Visualisierung](#visualisierung)
- [Speicherplatz und Aufbewahrung](#speicherplatz-und-aufbewahrung)
<!-- /TOC -->

## Was ist eine Zeitreihe?

Eine **Zeitreihe** ist eine Folge von Werten mit **Zeitstempel**:

```text
time                      item                   value
2026-10-01T06:58:12Z      iBad_Temperatur        20.4
2026-10-01T07:03:40Z      iBad_Licht             ON
2026-10-01T07:05:00Z      iBad_Temperatur        20.9
2026-10-01T07:21:15Z      iBad_Licht             OFF
```

| Eigenschaft | Bedeutung für die Speicherung |
| --- | --- |
| Daten kommen **fortlaufend** hinzu | Viele Schreibzugriffe, fast nie Änderungen |
| Abfragen beziehen sich auf **Zeiträume** | „Letzte 24 Stunden“, „Mittelwert pro Stunde“ |
| Alte Daten werden **weniger detailliert** gebraucht | Verdichten (*Downsampling*) und Löschen (*Retention*) |
| Datenmenge wächst **unbegrenzt** | Speicherplatz planen, besonders auf SD-Karten |

---

## Zeitreihendatenbanken

Relationale Datenbanken können Zeitreihen speichern, **Zeitreihendatenbanken** (TSDB) sind dafür aber optimiert: komprimierte Speicherung, schnelle Zeitbereichsabfragen, eingebaute Aggregationen und Aufbewahrungsrichtlinien.

| System | Typ | Hinweise |
| --- | --- | --- |
| **InfluxDB** | Zeitreihendatenbank | Weit verbreitet im Smart-Home-Umfeld, gut mit Grafana kombinierbar; Versionen unterscheiden sich in der Abfragesprache (InfluxQL, Flux, SQL) |
| **TimescaleDB** | PostgreSQL-Erweiterung | Zeitreihen mit vollem SQL |
| **rrd4j** | Round-Robin-Datenbank (Datei) | In openHAB eingebaut; feste Größe, verdichtet automatisch – ideal für Diagramme, **nicht** für exakte Historie |
| **JDBC** (MariaDB, PostgreSQL, SQLite …) | Relational | In openHAB über das JDBC-Add-on; einfach mit SQL auszuwerten |
| **MapDB** | Key-Value | In openHAB nur für den **letzten Zustand** (Wiederherstellung nach Neustart) |

Begriffe bei InfluxDB:

| Begriff | Bedeutung |
| --- | --- |
| **Measurement** | Vergleichbar mit einer Tabelle (in openHAB meist ein Item) |
| **Tag** | Indizierte Metadaten, z. B. Raum, Gerätetyp |
| **Field** | Der eigentliche Messwert |
| **Retention Policy / Bucket** | Wie lange Daten aufbewahrt werden |

---

## openHAB Persistence

### Wozu?

| Zweck | Beispiel |
| --- | --- |
| **Zustände nach Neustart wiederherstellen** | Ohne Persistence kennt openHAB nach einem Neustart den Zustand eines Fensterkontakts nicht. Eine Regel könnte eine Jalousie gegen das offene Fenster fahren. |
| **Diagramme** | Temperaturverlauf im Dashboard |
| **Regeln mit Historie** | „War das Licht in der letzten Stunde an?“, „Mittelwert der letzten 10 Minuten“ |
| **Auswertung und Vorhersage** | Gewohnheiten erkennen, Zustände vorhersagen |

### Konzept

* Mehrere **Persistence-Dienste** können **gleichzeitig** aktiv sein – z. B. MapDB für die Wiederherstellung, rrd4j für Diagramme, InfluxDB für die Langzeitauswertung.
* Ein **Standarddienst** wird für Diagramme und Regeln verwendet, wenn nichts anderes angegeben ist.
* Pro Dienst legt eine **Strategie** fest, **welche** Items **wann** gespeichert werden.

### Strategien

| Strategie | Speichert … |
| --- | --- |
| `everyChange` | … bei jeder **Änderung** des Zustands |
| `everyUpdate` | … bei jeder **Aktualisierung**, auch wenn der Wert gleich bleibt |
| **Cron-Strategien** | … zu festen Zeitpunkten, z. B. jede Minute oder stündlich |
| `restoreOnStartup` | Stellt den letzten Zustand beim Start **wieder her** |

Konfiguriert wird über die UI (*Settings → Persistence*) oder textbasiert in `$OPENHAB_CONF/persistence/<dienst>.persist`:

```text
// /etc/openhab/persistence/influxdb.persist
Strategies {
    everyMinute : "0 * * * * ?"
    everyHour   : "0 0 * * * ?"
    default = everyChange
}

Items {
    // all members of the group gHistory: on every change and additionally every hour
    gHistory*            : strategy = everyChange, everyHour
    // a single item with a fixed sampling rate
    iBad_Temperatur      : strategy = everyMinute
}
```

```text
// /etc/openhab/persistence/mapdb.persist
Strategies {
    default = everyUpdate
}

Items {
    * : strategy = everyUpdate, restoreOnStartup     // restore every item after a restart
}
```

> `gHistory*` bedeutet: alle **Mitglieder** der Gruppe. `gHistory` ohne Stern würde nur den Zustand der Gruppe selbst speichern.

> Die genaue Syntax (z. B. Filter, Alias) unterscheidet sich je nach openHAB-Version – im Zweifel die aktuelle Dokumentation unter *Configuration → Persistence* prüfen.

### Strategie für Auswertungen

Für spätere Analysen und Vorhersagen reicht `everyChange` allein oft **nicht**:

* Ein Schalter, der den ganzen Tag `OFF` ist, erzeugt **keinen einzigen** Eintrag. Ob er „aus“ war oder openHAB nicht lief, ist nicht unterscheidbar.
* Abhilfe: zusätzlich eine **feste Abtastrate** (z. B. stündlich oder minütlich) für die Items, die ausgewertet werden sollen – oder bei der Auswertung den letzten bekannten Zustand **fortschreiben** (*forward fill*).

---

## Daten auslesen

### Über die REST API

```bash
curl -s -H "Authorization: Bearer $OH_TOKEN" \
  "https://openhab.lab.local:8443/rest/persistence/items/iBad_Licht?serviceId=influxdb&starttime=2026-09-01T00:00:00.000Z&endtime=2026-10-01T00:00:00.000Z"
```

Die Antwort enthält die gespeicherten Datenpunkte (Zeitstempel als Millisekunden und Zustand). Für **große Zeiträume** ist die REST API langsam und paginiert; für Analysen ist der **direkte Zugriff auf die Datenbank** (z. B. mit dem InfluxDB-Client oder SQL) meist besser. Der Zugriff sollte dann mit einem **eigenen, nur lesenden** Datenbankbenutzer erfolgen.

### In Python auswerten

Sind die Daten geladen (z. B. als Liste von `(zeit, zustand)`), lassen sie sich mit **pandas** in ein gleichmäßiges Raster bringen – die Grundlage jeder Auswertung:

```python
import pandas as pd

# raw events as stored with 'everyChange'
events = pd.DataFrame({
    "time": pd.to_datetime(["2026-10-01 06:58", "2026-10-01 07:21",
                            "2026-10-01 19:40", "2026-10-01 22:55"]),
    "state": ["ON", "OFF", "ON", "OFF"],
}).set_index("time")

# regular 15-minute grid: carry the last known state forward
# (index = start of the interval, value = state at the end of the interval)
grid = (events["state"].eq("ON")
        .resample("15min").last()
        .ffill()
        .astype(bool))

# features for analysis / prediction
features = pd.DataFrame({
    "on": grid,
    "hour": grid.index.hour,
    "weekday": grid.index.weekday,
})
print(features.loc["2026-10-01 19:30":"2026-10-01 20:15"])
print("share of time on:", f"{features.on.mean():.0%}")
```

Wie man daraus eine Vorhersage baut – und welche Fehler man dabei vermeiden muss (chronologische Aufteilung, Baseline) –, zeigt [Machine-Learning-Grundlagen](../KI%20%26%20Sprachverarbeitung/Machine-Learning-Grundlagen.md).

---

## Visualisierung

| Werkzeug | Einsatz |
| --- | --- |
| **openHAB Charts** (MainUI, Sitemaps) | Schneller Blick auf einzelne Items |
| **Grafana** | Dashboards über viele Items, Vergleich mehrerer Zeiträume, Alarme |
| **Eigene Dashboards** | Mit einer Chart-Bibliothek (Chart.js, Plotly) über REST oder direkt aus der Datenbank |
| **Jupyter / Streamlit** | Explorative Auswertung, Prototypen |

---

## Speicherplatz und Aufbewahrung

* **Nicht alles speichern:** Nur Items, die wirklich ausgewertet werden, in die Langzeit-Datenbank (Gruppe `gHistory` o. Ä.).
* **Hohe Abtastraten** (jede Sekunde) nur, wenn nötig – sie füllen Speicher schnell und verschleißen SD-Karten.
* **Retention:** Detaildaten z. B. 90 Tage, verdichtete Stundenmittel dauerhaft.
* **Backup** der Datenbank einplanen (→ [Backup-Strategien](../Backup-Strategien/README.md)), denn Historie lässt sich nicht nachträglich erzeugen.
* **Datenschutz:** Zustandsverläufe zeigen, **wann jemand zu Hause ist, schläft, kocht** – das sind personenbezogene Daten. Zweck, Zugriff und Speicherdauer festlegen.

---
