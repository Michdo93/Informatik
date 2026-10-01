# 💸 Technische Schulden

**Technische Schulden** (*Technical Debt*) sind die zukünftigen Mehrkosten, die entstehen, weil man heute eine **schnelle statt einer sauberen Lösung** wählt. Wie bei einem Kredit bekommt man jetzt etwas (Zeit), zahlt aber später **mit Zinsen** zurück.

<!-- TOC -->
## Inhaltsverzeichnis

- [Herkunft der Metapher](#herkunft-der-metapher)
- [„Nichts hält so lange wie ein Provisorium“](#nichts-hält-so-lange-wie-ein-provisorium)
- [Arten technischer Schulden](#arten-technischer-schulden)
- [Schulden sichtbar machen](#schulden-sichtbar-machen)
  - [Im Code](#im-code)
  - [Im Projekt](#im-projekt)
- [Tilgen](#tilgen)
<!-- /TOC -->

## Herkunft der Metapher

Geprägt hat den Begriff **Ward Cunningham** (auch Erfinder des **Wikis**) im Jahr **1992**. Er wollte Managern einer Finanzsoftware-Firma erklären, warum Code überarbeitet werden muss, obwohl er „funktioniert“ – und wählte eine Sprache, die sie verstanden: Schulden und Zinsen.

| Finanzwelt | Software |
| --- | --- |
| Kredit aufnehmen | Schnelle Lösung / Workaround / Provisorium |
| Zinsen | Jede Änderung dauert länger, Fehler häufen sich |
| Tilgung | Refactoring, Aufräumen, Doku nachziehen |
| Überschuldung | Niemand traut sich mehr, den Code anzufassen; Neuschreiben wird nötig |

---

## „Nichts hält so lange wie ein Provisorium“

Typische Provisorien im Labor, die „nur kurz“ sein sollten – und Jahre bleiben:

| Provisorium | Zinsen | Saubere Lösung |
| --- | --- | --- |
| HTTP statt HTTPS „zum Testen“ | Passwörter im Klartext, später aufwendige Umstellung aller Clients | [Von Anfang an Zertifikate](../Best%20Practices/Zertifikate.md) |
| Dynamische IP „erst mal“ | Geräte nicht erreichbar nach Neustart, Regeln brechen | [Feste IP per DHCP-Reservierung](../Best%20Practices/DHCP.md) |
| MQTT ohne Passwort | Jeder im Netz kann Geräte steuern | [Passwort + ACL + TLS](../Best%20Practices/MQTT.md) |
| Passwort im Code | Landet in Git-History, nie wieder ganz entfernbar | Umgebungsvariablen, `.env` (nicht committen), Secret-Store |
| Einmal per Hand konfiguriert | Niemand weiß nach einem Jahr, wie es eingerichtet wurde | [Ansible](../Best%20Practices/Ansible.md), Doku |
| „Doku schreibe ich am Ende“ | Am Ende ist keine Zeit; Wissen geht mit dem Hiwi | [Doku von Anfang an](../Best%20Practices/Dokumentation.md) |
| Kein Backup „bis es fertig ist“ | Festplatte stirbt vorher | [Backup-Strategien](../Backup-Strategien/README.md) |

---

## Arten technischer Schulden

Martin Fowler unterscheidet im **Technical Debt Quadrant** zwei Achsen:

|  | **Rücksichtslos** | **Umsichtig** |
| --- | --- | --- |
| **Bewusst** | „Für Design haben wir keine Zeit.“ | „Wir liefern jetzt und kümmern uns um die Folgen.“ ✔️ (geplante Schulden) |
| **Versehentlich** | „Was ist Schichtenarchitektur?“ | „Jetzt wissen wir, wie wir es hätten machen sollen.“ (Lerneffekt) |

Bewusste, umsichtige Schulden sind **in Ordnung** – z. B. kurz vor einer Demo. Entscheidend ist, dass man sie **festhält** und **zurückzahlt**.

---

## Schulden sichtbar machen

### Im Code

```python
# TODO(michael): replace polling with MQTT subscription once firmware 2.x is installed
# FIXME: crashes if the beamer is unplugged during warm-up - see issue #17
# HACK: device sends temperature * 10 as string; remove when vendor fixes the API
# WORKAROUND: openHAB REST returns 'NULL' instead of null for undefined items
```

| Marker | Bedeutung |
| --- | --- |
| `TODO` | Noch zu erledigen |
| `FIXME` | Bekannter Fehler, muss behoben werden |
| `HACK` | Unsaubere Lösung, bewusst |
| `WORKAROUND` | Umgehung eines externen Problems |
| `XXX` | Achtung, gefährlich / dringend prüfen |

Viele IDEs und Linter listen diese Marker auf (VS Code „Todo Tree“, `ruff` Regel `FIX`, `TD`).

### Im Projekt

* [ ] Für jedes größere Provisorium ein **Issue** mit Label `tech-debt`.
* [ ] Abschnitt **„Known Issues / Limitations“** in der README.
* [ ] Bei Übergabe einer Abschlussarbeit oder Hiwi-Tätigkeit: **Liste offener Punkte** übergeben.

---

## Tilgen

* **Pfadfinder-Regel** (*Boy Scout Rule*): „Hinterlasse den Code sauberer, als du ihn vorgefunden hast.“ – bei jeder Änderung ein kleines Stück aufräumen.
* **Refactoring** in kleinen Schritten, abgesichert durch Tests.
* Regelmäßig **feste Zeit** für Aufräumarbeiten einplanen (z. B. jeder fünfte Tag).
* Erst messen/verstehen, dann umbauen – **nicht** aus Prinzip alles neu schreiben.

---
