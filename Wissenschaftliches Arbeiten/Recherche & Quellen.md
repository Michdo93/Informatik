# 🔬 Recherche & Quellen

Wer im Labor eine **Projekt- oder Abschlussarbeit** schreibt, muss nicht nur programmieren, sondern auch **wissenschaftlich arbeiten**: Quellen finden, bewerten, korrekt zitieren und die eigene Arbeit sauber aufbauen. Dieses Kapitel gibt eine praktische Einführung.

<!-- TOC -->
## Inhaltsverzeichnis

- [Gute Quellen finden](#gute-quellen-finden)
- [Quellen bewerten](#quellen-bewerten)
- [Zitieren](#zitieren)
  - [Zitierstile](#zitierstile)
  - [Literaturverwaltung](#literaturverwaltung)
- [KI-Werkzeuge in Abschlussarbeiten](#ki-werkzeuge-in-abschlussarbeiten)
- [Aufbau einer Abschlussarbeit (typisch)](#aufbau-einer-abschlussarbeit-typisch)
- [Werkzeuge zum Schreiben](#werkzeuge-zum-schreiben)
- [Checkliste](#checkliste)
<!-- /TOC -->

## Gute Quellen finden

| Quelle | Wofür | Hinweis |
| --- | --- | --- |
| **Google Scholar** | Wissenschaftliche Artikel finden | Zeigt Zitationszahlen und Versionen |
| **IEEE Xplore, ACM Digital Library** | Informatik-Fachliteratur | Über die Hochschulbibliothek meist kostenfrei |
| **SpringerLink, ScienceDirect** | Bücher und Journals | Zugang über die HFU |
| **arXiv** | Preprints (noch nicht begutachtet) | Aktuell, aber **nicht peer-reviewed** – vorsichtig einordnen |
| **Hochschulbibliothek / Fernleihe** | Bücher, Normen | VPN/Shibboleth für Zugriff von zu Hause |
| **Herstellerdoku, Standards (RFC, W3C)** | Technische Details | Primärquelle, sehr zuverlässig |
| **Projekt-Dokumentationen (openHAB, ROS …)** | Praktische Umsetzung | Versionsabhängig – Stand angeben |

**Weniger geeignet** als alleinige Quelle: Blogs, Foren, Stack Overflow, Wikipedia, KI-Chatbots. Sie sind gut zum **Einstieg und Verstehen**, aber in einer wissenschaftlichen Arbeit nur mit Vorsicht und selten zitierfähig. Wikipedia führt oft zu guten Primärquellen – diese dann direkt zitieren.

---

## Quellen bewerten

Nicht alles, was gefunden wird, ist verlässlich. Prüffragen:

* **Wer?** Autorin/Autor, Institution – ausgewiesen und einschlägig?
* **Wo erschienen?** Peer-reviewed Journal/Konferenz, Buch, Preprint, Blog?
* **Wann?** Aktuell genug? In der IT veralten Details schnell (Versionen, Bibliotheken).
* **Warum?** Sachlich oder werblich/interessengeleitet?
* **Belegt?** Hat die Quelle selbst Belege, sind Aussagen nachvollziehbar?
* **Konsens?** Sagen andere Quellen Ähnliches, oder steht die Aussage allein?

> Für technische „Fakten“ aus Blogs oder von KI-Werkzeugen gilt: **gegenprüfen** an der Primärquelle (Doku, Standard, Quellcode), bevor man sie übernimmt.

---

## Zitieren

**Jede** fremde Aussage, Zahl, Abbildung oder Idee muss belegt werden – sonst ist es ein **Plagiat**. Zitiert wird

* **direkt** (wörtlich, in Anführungszeichen) – sparsam, nur wenn die genaue Formulierung zählt,
* **indirekt** (paraphrasiert, in eigenen Worten) – der Normalfall, trotzdem mit Quelle.

### Zitierstile

In der Informatik ist der **IEEE-Stil** verbreitet: nummerierte Quellen in eckigen Klammern `[1]`, `[2]`, das Literaturverzeichnis in der Reihenfolge des Auftretens.

```text
Text ... wie in [1] gezeigt. Weitere Ansätze [2], [3] verfolgen ...

[1] A. Autor, "Titel des Artikels," Zeitschrift, Bd. 12, Nr. 3, S. 45–67, 2023.
[2] B. Autorin, Titel des Buches, 2. Aufl. Ort: Verlag, 2021.
```

Andere Stile sind **APA**, **Harvard** (Autor-Jahr: „(Autor, 2023)“), **Chicago**. **Wichtig:** Den von der Betreuung bzw. der Prüfungsordnung geforderten Stil verwenden und **einheitlich** durchhalten.

### Literaturverwaltung

Quellen von Anfang an sammeln – nicht am Ende mühsam rekonstruieren. Werkzeuge:

| Werkzeug | Hinweise |
| --- | --- |
| **Zotero** | Kostenlos, Open Source, Browser-Integration, BibTeX-Export – gute Standardwahl |
| **JabRef** | Open Source, speziell für **BibTeX** (LaTeX) |
| **Citavi** | An vielen Hochschulen lizenziert (Windows) |
| **Mendeley** | Verbreitet, kommerziell |

Für **LaTeX** verwaltet man Quellen als **BibTeX**-Datei (`.bib`) und zitiert mit `\cite{schluessel}`; das Literaturverzeichnis erzeugt **biblatex**/**biber** automatisch im gewählten Stil.

```bibtex
@inproceedings{mueller2023,
  author    = {Müller, Anna and Schmidt, Ben},
  title      = {Interoperability of Smart Home Devices with openHAB},
  booktitle = {Proc. IEEE Int. Conf. on Consumer Electronics},
  year       = {2023},
  pages      = {45--50},
}
```

---

## KI-Werkzeuge in Abschlussarbeiten

KI-Chatbots dürfen beim Schreiben helfen (Ideen, Formulierungen, Code-Verständnis), aber:

* **Prüfungsordnung/Betreuung fragen**, was erlaubt ist und wie es **kenntlich** gemacht werden muss. Viele Hochschulen verlangen eine Erklärung über den Einsatz von KI-Hilfsmitteln.
* KI **erfindet Quellen** („halluziniert“). Jede von einer KI genannte Referenz **selbst prüfen** – gibt es sie wirklich, sagt sie das Behauptete?
* KI ersetzt **keine** eigene Recherche und kein eigenes Verständnis. Die inhaltliche Verantwortung liegt bei der Autorin/dem Autor.
* Code und Texte von KI genauso bewerten wie fremde Quellen: verstehen, prüfen, testen.

---

## Aufbau einer Abschlussarbeit (typisch)

| Kapitel | Inhalt |
| --- | --- |
| **Einleitung** | Motivation, Problemstellung, Ziel, Aufbau |
| **Grundlagen / Stand der Technik** | Begriffe, verwandte Arbeiten, eingesetzte Technologien |
| **Anforderungsanalyse** | Funktionale und nicht-funktionale Anforderungen |
| **Konzept / Architektur** | Entwurf, Entscheidungen, Alternativen abgewogen |
| **Implementierung** | Umsetzung, wichtige Details, Probleme und Lösungen |
| **Evaluation** | Tests, Messungen, Bewertung gegen die Anforderungen |
| **Fazit & Ausblick** | Zusammenfassung, Grenzen, mögliche Weiterentwicklung |

Dazu: Titelblatt, Abstract/Zusammenfassung, Inhaltsverzeichnis, Abbildungs-/Tabellenverzeichnis, Literaturverzeichnis, ggf. Anhang und die **eidesstattliche Erklärung**.

> **Stand der Technik ernst nehmen:** Gerade bei Laborthemen (openHAB, Roboter, Sprachassistenten) gibt es viel Vorarbeit – eigene und fremde. Eine gute Arbeit ordnet sich darin ein und grenzt sich ab, statt bei null zu beginnen.

---

## Werkzeuge zum Schreiben

| Werkzeug | Stärken | Hinweise |
| --- | --- | --- |
| **LaTeX** (TeX Live, Overleaf) | Formelsatz, Referenzen, stabile Formatierung bei langen Dokumenten | Höhere Einstiegshürde; im Labor verbreitet |
| **Word / LibreOffice** | Einfacher Einstieg, Kommentare | Bei langen Arbeiten mit vielen Referenzen weniger komfortabel |
| **Markdown + Pandoc** | Einfach, versionierbar | Für formale Arbeiten meist zu begrenzt |

Für Diagramme eignen sich **Mermaid**, **PlantUML**, **draw.io/diagrams.net** oder **TikZ** (LaTeX). Abbildungen immer mit **Quelle** bzw. „eigene Darstellung“ kennzeichnen.

---

## Checkliste

* [ ] Quellen von Anfang an in einem Literaturverwaltungsprogramm gesammelt
* [ ] Zitierstil mit der Betreuung abgestimmt und **einheitlich** verwendet
* [ ] Jede fremde Aussage/Zahl/Abbildung belegt (kein Plagiat)
* [ ] Quellen auf Verlässlichkeit und Aktualität geprüft
* [ ] Einsatz von KI-Werkzeugen geklärt und (falls gefordert) kenntlich gemacht
* [ ] Stand der Technik recherchiert und die eigene Arbeit darin eingeordnet
* [ ] Eidesstattliche Erklärung nicht vergessen

---
