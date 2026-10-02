# 🩹 Workarounds & Hacks

In der Praxis passt selten alles perfekt zusammen: Browser unterstützen ein Feature nicht, eine Bibliothek hat einen Bug, ein Gerät spricht ein exotisches Protokoll. Dann helfen **Workarounds** und **Hacks**. Dieser Ordner erklärt die gängigen Techniken – und wann man sie besser **nicht** einsetzt.

<!-- TOC -->
## Inhaltsverzeichnis

- [Übersicht](#übersicht)
- [Grundregeln für Workarounds](#grundregeln-für-workarounds)
<!-- /TOC -->

## Übersicht

| Dokument | Inhalt |
| --- | --- |
| [Begriffe: Workaround, Hack, Kludge & Co.](Begriffe.md) | Was ist ein Workaround, Hack, Kludge, Quick Fix, Hotfix, Monkey Patch? Woher kommen die Wörter? |
| [Polyfill & Shim](Polyfill%20%26%20Shim.md) | Fehlende Funktionen in alten Browsern/Laufzeiten nachrüsten – inkl. Herkunft des Namens |
| [Feature Detection & Browser-Weichen](Feature%20Detection%20%26%20Browser-Weichen.md) | Feature Detection statt User-Agent-Sniffing, Vendor-Prefixe, Graceful Degradation vs. Progressive Enhancement |
| [Monkey Patching](Monkey%20Patching.md) | Fremden Code zur Laufzeit verändern – mächtig und gefährlich |
| [Cross-Plattform-Tricks](Cross-Plattform-Tricks.md) | Pfade, Zeilenenden, Encoding, Shebangs, Linux/Windows/macOS-Unterschiede |
| [Reverse Engineering](Reverse%20Engineering.md) | Geräte ohne offene Schnittstelle verstehen: rechtlicher Rahmen, Wireshark, BLE-Snoop, serielle Schnittstellen, APKs mit jadx/apktool, Firmware, Dokumentation |
| [Technische Schulden](Technische%20Schulden.md) | Warum jedes Provisorium Zinsen kostet und wie man Workarounds sauber dokumentiert |

---

## Grundregeln für Workarounds

1. **Erst die Ursache verstehen**, dann umgehen. Ein Workaround ohne Verständnis ist Raten.
2. **Kommentieren**, warum es den Workaround gibt – mit Link auf Issue/Bugreport/Doku:

   ```python
   # WORKAROUND: paho-mqtt 1.x ignores keepalive < 5 s, see https://github.com/...
   # Remove when upgrading to 2.x (tracked in issue #42).
   KEEPALIVE = max(configured_keepalive, 5)
   ```

3. **Eingrenzen:** Workaround nur dort, wo er nötig ist (Version prüfen, Feature prüfen).
4. **Ablaufdatum festlegen:** Ticket/Issue anlegen, damit er wieder entfernt wird.
5. **Upstream melden:** Bug beim Hersteller/Projekt melden – vielleicht profitieren alle.
6. Bewusst bleiben: **„Nichts hält so lange wie ein Provisorium.“** (→ [Technische Schulden](Technische%20Schulden.md))

---
