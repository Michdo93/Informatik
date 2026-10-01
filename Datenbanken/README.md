# 🗄️ Datenbanken

Fast jedes Projekt speichert Daten – Benutzer, Geräte, Trainingsdaten, Messwerte. Dieser Ordner erklärt, wie man Daten sinnvoll strukturiert und wie man mit den Zeitreihen eines Smart Homes umgeht.

<!-- TOC -->
## Inhaltsverzeichnis

- [Übersicht](#übersicht)
- [Verwandte Dokumente](#verwandte-dokumente)
<!-- /TOC -->

## Übersicht

| Dokument | Inhalt |
| --- | --- |
| [Relationale Modellierung](Relationale%20Modellierung.md) | Tabellen, Schlüssel, Beziehungen 1:1, 1:n und n:m (Zwischentabelle am Beispiel Synonyme), Normalisierung, Dateien vs. BLOB, SQLite-Beispiel, Datenbanksysteme, ORM |
| [Zeitreihen & openHAB Persistence](Zeitreihen%20%26%20openHAB%20Persistence.md) | Zeitreihendatenbanken (InfluxDB, rrd4j, JDBC), Persistence-Strategien, `restoreOnStartup`, Daten per REST auslesen, Auswertung mit pandas, Aufbewahrung und Datenschutz |

---

## Verwandte Dokumente

* [Backup-Strategien → Export & Import](../Backup-Strategien/Export%20%26%20Import.md) – Datenbanken sichern und wiederherstellen
* [Zugriffskontrolle → ACL](../Zugriffskontrolle/ACL.md) – Rechte in Datenbanken
* [Machine-Learning-Grundlagen](../KI%20%26%20Sprachverarbeitung/Machine-Learning-Grundlagen.md) – Daten für Vorhersagen nutzen

---
