# 🐞 Debugging

**Debugging** ist das systematische Finden und Beheben von Fehlern. Es ist eine Fähigkeit, die man lernen kann – und die mehr mit **wissenschaftlichem Arbeiten** als mit Glück zu tun hat.

<!-- TOC -->
## Inhaltsverzeichnis

- [Übersicht](#übersicht)
- [Warum heißt es „Debugging“?](#warum-heißt-es-debugging)
- [Die drei wichtigsten Regeln](#die-drei-wichtigsten-regeln)
<!-- /TOC -->

## Übersicht

| Dokument | Inhalt |
| --- | --- |
| [Vorgehen beim Debugging](Vorgehen%20beim%20Debugging.md) | Systematische Methode, Fehler reproduzieren, Hypothesen, Bisektion, Rubber Duck, typische Fehlerklassen, gute Fehlerberichte |
| [Werkzeuge](Werkzeuge.md) | Print vs. Logging vs. Debugger, Breakpoints, `pdb`, `gdb`, Browser-DevTools, Netzwerk-Tools, Tracing |
| [Remote-Debugging](Remote-Debugging.md) | Programme auf Raspberry Pi, Server, Container oder Roboter vom eigenen Rechner aus debuggen: `debugpy`, `gdbserver`, Node `--inspect`, JDWP, SSH-Tunnel |

---

## Warum heißt es „Debugging“?

Ein **Bug** ist ein Fehler, **De-bugging** das Entfernen der „Käfer“. Die berühmte Motte, die 1947 im Relais des Harvard Mark II gefunden wurde, hat den Begriff **nicht erfunden** – *bug* war schon bei Edison für technische Macken üblich –, ihn aber populär gemacht. Die ganze Geschichte steht unter [Namensherkunft & Analogien](../Begriffe%20%26%20Herkunft/Namensherkunft%20%26%20Analogien.md).

---

## Die drei wichtigsten Regeln

1. **Reproduzieren, bevor man repariert.** Ein Fehler, den man nicht auslösen kann, kann man auch nicht als behoben nachweisen.
2. **Eine Änderung zur Zeit.** Wer fünf Dinge gleichzeitig ändert, weiß hinterher nicht, was geholfen hat.
3. **Die Fehlermeldung vollständig lesen.** Der Stacktrace sagt meist genau, **wo** es knallt – die unterste Zeile in Python zeigt den Fehler, die Zeilen darüber den Weg dorthin.

---
