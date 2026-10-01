# 🧰 Debugging-Werkzeuge

Vom einfachen `print()` bis zum Netzwerk-Mitschnitt: Welches Werkzeug passt zu welchem Problem?

<!-- TOC -->
## Inhaltsverzeichnis

- [Print, Logging oder Debugger?](#print-logging-oder-debugger)
- [Debugger-Grundbegriffe](#debugger-grundbegriffe)
- [Python: pdb und IDE](#python-pdb-und-ide)
- [C/C++: gdb](#cc-gdb)
- [Browser: DevTools](#browser-devtools)
- [Netzwerk und System](#netzwerk-und-system)
- [ROS / Robotik](#ros--robotik)
<!-- /TOC -->

## Print, Logging oder Debugger?

| Werkzeug | Gut für | Nachteile |
| --- | --- | --- |
| **`print()`** | Schneller Blick auf einen Wert | Muss wieder entfernt werden, keine Zeitstempel, keine Level |
| **Logging** | Dauerhafte Nachvollziehbarkeit, Produktivsysteme, sporadische Fehler | Muss vorher eingebaut sein |
| **Debugger** | Programmzustand interaktiv untersuchen, Schritt für Schritt | Verändert Timing (Heisenbugs), bei verteilten Systemen schwierig |
| **Tracing** | Abläufe über Prozess- und Rechnergrenzen | Aufwendiger Aufbau |

Faustregel: **Logging immer**, **Debugger bei Bedarf**, **print nur kurz** (und nie committen). Siehe auch [Fehlermeldungen, Logging & Exceptions](../Code-Formatierung/Fehlermeldungen%2C%20Logging%20%26%20Exceptions.md).

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s %(levelname)-8s %(name)s: %(message)s",
)
log = logging.getLogger("beamer")

log.debug("sending %r", b"\r*pow=on#\r")
log.warning("no answer after %d s, retrying", 5)
```

---

## Debugger-Grundbegriffe

| Begriff | Bedeutung |
| --- | --- |
| **Breakpoint** | Programm hält an dieser Zeile an |
| **Conditional Breakpoint** | Hält nur an, wenn eine Bedingung erfüllt ist (`temperature > 30`) |
| **Logpoint** | Gibt etwas aus, ohne anzuhalten (wie `print`, aber ohne Codeänderung) |
| **Step Over** | Nächste Zeile, Funktionsaufrufe werden ausgeführt, aber nicht betreten |
| **Step Into** | In die aufgerufene Funktion hineinspringen |
| **Step Out** | Bis zum Ende der aktuellen Funktion laufen |
| **Continue** | Weiterlaufen bis zum nächsten Breakpoint |
| **Watch** | Ausdruck, der bei jedem Halt neu ausgewertet wird |
| **Call Stack** | Die Kette der Funktionsaufrufe bis zur aktuellen Stelle |
| **Post-Mortem** | Debuggen **nach** dem Absturz am Ort der Exception |
| **Core Dump** | Speicherabbild eines abgestürzten Prozesses (C/C++), später mit `gdb` analysierbar |

---

## Python: pdb und IDE

```python
def average(values):
    breakpoint()            # Python >= 3.7: stops here and opens pdb
    return sum(values) / len(values)
```

| pdb-Befehl | Wirkung |
| --- | --- |
| `n` | next (Step Over) |
| `s` | step (Step Into) |
| `c` | continue |
| `p ausdruck` / `pp` | Wert ausgeben / schön ausgeben |
| `l` / `ll` | Quelltext anzeigen |
| `w` | Call Stack (where) |
| `u` / `d` | Im Stack hoch / runter |
| `b datei:zeile` | Breakpoint setzen |
| `q` | beenden |

```bash
python -m pdb skript.py             # start under the debugger
python -m pdb -c continue skript.py # run, stop only on exception (post-mortem)
```

In **VS Code** oder **PyCharm** geht das komfortabler per Klick auf den Zeilenrand. Die Konfiguration steht in `.vscode/launch.json`.

---

## C/C++: gdb

```bash
gcc -g -O0 -o sensor sensor.c        # -g: debug symbols, -O0: no optimization
gdb ./sensor
```

| gdb-Befehl | Wirkung |
| --- | --- |
| `run args` | Programm starten |
| `break main` / `b datei.c:42` | Breakpoint |
| `next` / `step` / `finish` | Step Over / Into / Out |
| `print var` / `p *ptr` | Wert anzeigen |
| `bt` | Backtrace (Call Stack) |
| `watch var` | Anhalten, wenn sich `var` ändert |
| `info locals` | Lokale Variablen |

**Speicherfehler finden:** `valgrind ./sensor` oder beim Kompilieren `-fsanitize=address,undefined` (AddressSanitizer) – findet Pufferüberläufe und Use-after-free, die sonst nur „manchmal“ abstürzen.

---

## Browser: DevTools

`F12` öffnet die Entwicklerwerkzeuge in jedem Browser:

* **Console:** Fehlermeldungen, `console.log`, JavaScript ausführen.
* **Sources:** Breakpoints in JavaScript, auch in Event-Listenern.
* **Network:** Jede Anfrage mit Status, Headern, Antwort und Zeit – unverzichtbar für REST-APIs (z. B. openHAB-REST).
* **Elements:** HTML/CSS live verändern.
* **Application:** Cookies, LocalStorage, Service Worker.

---

## Netzwerk und System

| Werkzeug | Frage | Beispiel |
| --- | --- | --- |
| `ping` | Ist der Host erreichbar? | `ping 192.168.10.21` |
| `nc` / `telnet` | Ist der Port offen? | `nc -zv 192.168.10.21 8883` |
| `ss` | Welche Ports lauschen hier? | `sudo ss -tlnp` |
| `curl -v` | Was antwortet die HTTP-API? | `curl -v https://openhab.lab:8443/rest/items` |
| `openssl s_client` | Stimmt das Zertifikat? | `openssl s_client -connect host:8883 -CAfile ca.crt` |
| `mosquitto_sub` | Was läuft über MQTT? | `mosquitto_sub -h broker -t '#' -v` |
| `tcpdump` / **Wireshark** | Was geht wirklich über die Leitung? | `sudo tcpdump -i eth0 port 1883 -w dump.pcap` |
| `strace` | Welche Systemaufrufe macht ein Programm? (Dateien, Netzwerk) | `strace -f -e trace=openat,connect python app.py` |
| `journalctl` | Was sagen die Dienste? | `journalctl -u mosquitto -f` |
| `htop`, `iotop` | Wer frisst CPU/RAM/IO? | – |

---

## ROS / Robotik

* `ros2 topic echo /scan`, `ros2 topic hz /odom` – kommen Daten an, in welcher Rate?
* `rqt_graph` – welche Nodes sind wie verbunden?
* `ros2 bag record` – Daten aufzeichnen und später **reproduzierbar** abspielen (ideal für sporadische Fehler).
* **RViz / Foxglove** – Sensordaten visualisieren statt Zahlen anzustarren.

---
