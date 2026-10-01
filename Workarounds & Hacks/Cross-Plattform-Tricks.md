# 🖥️ Cross-Plattform-Tricks

Code, der auf dem eigenen Laptop läuft, scheitert oft auf dem Raspberry Pi, dem Windows-Rechner einer Kommilitonin oder im Docker-Container. Die Ursachen sind fast immer dieselben – hier die Klassiker und ihre Lösungen.

<!-- TOC -->
## Inhaltsverzeichnis

- [Die häufigsten Stolperfallen](#die-häufigsten-stolperfallen)
- [Pfade](#pfade)
- [Zeilenenden – der Klassiker](#zeilenenden--der-klassiker)
- [Encoding](#encoding)
- [Betriebssystem erkennen – wenn es sein muss](#betriebssystem-erkennen--wenn-es-sein-muss)
- [Architektur: x86 vs. ARM](#architektur-x86-vs-arm)
- [Container als ultimativer Workaround](#container-als-ultimativer-workaround)
<!-- /TOC -->

## Die häufigsten Stolperfallen

| Thema | Linux / macOS | Windows | Lösung |
| --- | --- | --- | --- |
| Pfadtrenner | `/` | `\` (auch `/` meist ok) | `pathlib`, `os.path.join` |
| Zeilenende | `LF` (`\n`) | `CRLF` (`\r\n`) | `.gitattributes`, Editor-Einstellung |
| Groß-/Kleinschreibung in Dateinamen | Linux: unterschieden; macOS: meist nicht | nicht unterschieden | Konsequent kleinschreiben, Imports exakt |
| Encoding | UTF-8 | historisch cp1252, zunehmend UTF-8 | `encoding="utf-8"` immer angeben |
| Shebang / Ausführbarkeit | `#!/usr/bin/env python3`, `chmod +x` | wird ignoriert, Dateiendung zählt | `python script.py` dokumentieren |
| Home-Verzeichnis | `/home/user`, `~` | `C:\Users\user` | `Path.home()` |
| Temp-Verzeichnis | `/tmp` | `%TEMP%` | `tempfile` |
| Python-Aufruf | `python3` | `py` / `python` | venv nutzen, dann immer `python` |
| Verbotene Dateinamen | nur `/` und NUL | `CON`, `NUL`, `AUX`, `:`, `?`, `*`, `|`, `<`, `>`, `"` | Keine Sonderzeichen, keine Leerzeichen |
| Maximale Pfadlänge | ~4096 | historisch 260 Zeichen | Flache Strukturen, kurze Namen |

---

## Pfade

```python
from pathlib import Path

base = Path(__file__).resolve().parent          # directory of this script
config = base / "config" / "local.yaml"         # works on every OS
backup_dir = Path.home() / "backups"
backup_dir.mkdir(parents=True, exist_ok=True)

print(config.exists(), config.suffix, config.stem)
```

> **Tipp:** Pfade **relativ zum Skript** bilden, nie relativ zum aktuellen Arbeitsverzeichnis. Cron, systemd und openHAB-Exec starten Programme oft in `/` oder im Home-Verzeichnis (→ [Cron & systemd-Timer](../Linux%20%26%20Werkzeuge/Cron%20%26%20systemd-Timer.md)).

---

## Zeilenenden – der Klassiker

Ein unter Windows bearbeitetes Shell-Skript enthält `CRLF`. Auf Linux kommt dann:

```text
/usr/bin/env: 'bash\r': No such file or directory
```

**Herkunft:** `CR` (Carriage Return, Wagenrücklauf) und `LF` (Line Feed, Zeilenvorschub) stammen von der **Schreibmaschine** bzw. dem Fernschreiber: Der Druckkopf fährt zurück an den Anfang (CR), das Papier rückt eine Zeile weiter (LF). Windows behielt beides, Unix nur LF, alte Macs nur CR.

**Lösung im Repo** – Datei `.gitattributes`:

```gitattributes
* text=auto eol=lf
*.bat text eol=crlf
*.ps1 text eol=crlf
*.png binary
*.jpg binary
```

Reparatur einzelner Dateien: `dos2unix skript.sh` oder `sed -i 's/\r$//' skript.sh`.

---

## Encoding

```python
# Always specify the encoding - the default differs between systems
with open("notes.md", "w", encoding="utf-8") as f:
    f.write("Umlaute: äöüß, Grad: °C\n")
```

* Python ≥ 3.15 wird UTF-8 als Standard verwenden (PEP 686); bis dahin **immer explizit** angeben.
* Unter Windows hilft die Umgebungsvariable `PYTHONUTF8=1`.
* CSV-Dateien für Excel: `encoding="utf-8-sig"` (mit BOM), sonst erkennt Excel Umlaute oft falsch.

---

## Betriebssystem erkennen – wenn es sein muss

```python
import platform
import sys

if sys.platform.startswith("linux"):
    serial_port = "/dev/ttyUSB0"
elif sys.platform == "win32":
    serial_port = "COM3"
elif sys.platform == "darwin":
    serial_port = "/dev/tty.usbserial-0001"

print(platform.system(), platform.machine())    # e.g. Linux aarch64 (Raspberry Pi 64-bit)
```

Besser noch: solche Werte in eine **Konfigurationsdatei** auslagern statt im Code zu verzweigen.

---

## Architektur: x86 vs. ARM

Raspberry Pi und Apple Silicon sind **ARM** (`aarch64`/`arm64`), die meisten PCs und Server **x86-64** (`amd64`).

* Python-Pakete mit C-Erweiterungen brauchen passende **Wheels** – sonst wird beim `pip install` kompiliert (langsam, braucht Compiler).
* Docker-Images müssen für die Zielarchitektur existieren → **Multi-Arch-Images** mit `docker buildx build --platform linux/amd64,linux/arm64`.
* Eigene C/C++-Programme für den Pi auf dem PC bauen → **Cross-Compiling** (→ [Compiler & Build](../Compiler%20%26%20Build/README.md)).

---

## Container als ultimativer Workaround

„Bei mir läuft's!“ – „Dann liefern wir eben deinen Rechner aus.“ Genau das ist (vereinfacht) die Idee von **Docker**: Programm **plus Laufzeitumgebung** werden gemeinsam ausgeliefert. Für Laborprojekte heißt das:

* [ ] `Dockerfile` oder `compose.yaml` im Repo, wenn das Projekt Dienste braucht.
* [ ] Alternativ: `pyproject.toml`/`requirements.txt` mit **festen Versionen** + Python-Version in der README.
* [ ] Dev-Container (`.devcontainer/`) für VS Code, damit alle dieselbe Umgebung haben.

---
