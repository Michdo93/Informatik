# 📜 Bash-Skripte

Shell-Skripte sind der Klebstoff im Labor: Backups anstoßen, Geräte abfragen, Dienste neu starten, Konfigurationen verteilen. Dieses Kapitel fasst zusammen, was man für **robuste** Skripte braucht – und welche Fallen man kennen sollte.

<!-- TOC -->
## Inhaltsverzeichnis

- [Shell, sh und Bash](#shell-sh-und-bash)
- [Das Grundgerüst](#das-grundgerüst)
- [Shebang](#shebang)
- [Strict Mode: set -euo pipefail](#strict-mode-set--euo-pipefail)
- [Variablen und Quoting](#variablen-und-quoting)
  - [Sonderparameter](#sonderparameter)
- [Bedingungen](#bedingungen)
- [Schleifen](#schleifen)
- [Funktionen](#funktionen)
- [Ein- und Ausgabe umleiten](#ein--und-ausgabe-umleiten)
- [Exit-Codes](#exit-codes)
- [Aufräumen mit trap](#aufräumen-mit-trap)
- [Typische Fallen](#typische-fallen)
- [Werkzeuge](#werkzeuge)
- [Wann lieber Python?](#wann-lieber-python)
- [Checkliste](#checkliste)
<!-- /TOC -->

## Shell, sh und Bash

| Begriff | Bedeutung |
| --- | --- |
| **Shell** | Kommandozeilen-Interpreter, die „Schale“ um den Kernel (→ [Namensherkunft](../Begriffe%20%26%20Herkunft/Namensherkunft%20%26%20Analogien.md)) |
| **sh** | POSIX-Standard-Shell; unter Debian/Ubuntu/Raspberry Pi OS ist `/bin/sh` die schlanke **dash** |
| **Bash** | *Bourne Again Shell* – Standard-Login-Shell der meisten Linux-Systeme, mit vielen Erweiterungen gegenüber `sh` |
| **zsh** | Standard-Shell unter macOS, weitgehend Bash-kompatibel |

> **Wichtig:** Bash-Features (Arrays, `[[ … ]]`, `{1..5}`, `$(< datei)`) funktionieren **nicht** in `sh`/dash. Ein Skript mit `#!/bin/sh` oder gestartet mit `sh skript.sh` bricht dann mit seltsamen Fehlern ab. Cron und manche Programme verwenden `/bin/sh`!

---

## Das Grundgerüst

```bash
#!/usr/bin/env bash
#
# backup_openhab.sh - create a compressed backup of the openHAB configuration
#
# Usage: backup_openhab.sh [-n] <target-dir>
#   -n  dry run (show what would be done)

set -euo pipefail        # strict mode, see below
IFS=$'\n\t'

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"   # directory of this script
TIMESTAMP="$(date +%F_%H-%M-%S)"
readonly SCRIPT_DIR TIMESTAMP     # assign first, then make read-only (ShellCheck SC2155)

log() { printf '%s [%s] %s\n' "$(date +%T)" "$1" "$2" >&2; }
die() { log ERROR "$1"; exit 1; }

main() {
    local dry_run=false
    while getopts ":n" opt; do
        case "$opt" in
            n) dry_run=true ;;
            *) die "unknown option -$OPTARG" ;;
        esac
    done
    shift $((OPTIND - 1))

    [[ $# -eq 1 ]] || die "usage: $(basename "$0") [-n] <target-dir>"
    local target="$1"
    [[ -d "$target" ]] || die "target directory '$target' does not exist"

    local archive="$target/openhab_$TIMESTAMP.tar.gz"
    log INFO "creating $archive (script in $SCRIPT_DIR)"
    if $dry_run; then
        log INFO "dry run - nothing written"
    else
        tar -czf "$archive" -C /etc openhab
    fi
    log INFO "done"
}

main "$@"
```

Ausführbar machen und starten:

```bash
chmod +x backup_openhab.sh
./backup_openhab.sh /mnt/nas/backups
```

---

## Shebang

Die erste Zeile legt fest, **welcher Interpreter** das Skript ausführt:

| Shebang | Bedeutung |
| --- | --- |
| `#!/usr/bin/env bash` | Bash aus dem `PATH` – **empfohlen**, funktioniert auch, wenn Bash woanders liegt |
| `#!/bin/bash` | Fester Pfad – auf Linux praktisch immer vorhanden |
| `#!/bin/sh` | POSIX-Shell (dash) – nur ohne Bash-Features |
| `#!/usr/bin/env python3` | Funktioniert genauso für Python-Skripte |

Der Name kommt von **sharp** (`#`) + **bang** (`!`).

---

## Strict Mode: set -euo pipefail

| Option | Wirkung | Ohne sie … |
| --- | --- | --- |
| `-e` | Skript bricht beim **ersten fehlgeschlagenen Befehl** ab | … läuft es nach einem Fehler einfach weiter (z. B. löscht alte Backups, obwohl das neue fehlschlug) |
| `-u` | Abbruch bei **nicht gesetzten Variablen** | … wird `rm -rf "$DIR/"*` mit leerem `$DIR` zu `rm -rf /*` |
| `-o pipefail` | Eine Pipe schlägt fehl, wenn **irgendein** Teil fehlschlägt | … zählt nur der letzte Befehl: `mysqldump … \| gzip` „klappt“ auch, wenn der Dump scheitert |
| `-x` | Jeden Befehl vor der Ausführung ausgeben | (zum **Debuggen**: `bash -x skript.sh`) |

Fehler, die erwartet sind, explizit abfangen:

```bash
if ! ping -c1 -W2 "$host" >/dev/null; then
    log WARN "$host not reachable"
fi
grep -q "pattern" file || true      # no match is fine here
```

---

## Variablen und Quoting

```bash
name="beamer"                 # no spaces around '='!
port=8883
echo "Device: $name, port: ${port}"
echo "Backup: ${name}_backup.tar"     # braces needed when text follows

readonly CONFIG="/etc/lab/app.conf"   # constant
local count=0                         # only inside functions

echo "${MQTT_HOST:-localhost}"        # default if unset/empty
: "${MQTT_PASSWORD:?MQTT_PASSWORD must be set}"   # abort if missing
```

**Immer doppelte Anführungszeichen** um Variablen: `"$datei"`. Sonst zerfallen Werte mit Leerzeichen in mehrere Wörter, und `*` wird zu Dateinamen expandiert.

| Schreibweise | Bedeutung |
| --- | --- |
| `"…"` | Variablen und `$(…)` werden ersetzt |
| `'…'` | Alles wörtlich, keine Ersetzung |
| `$(befehl)` | Ausgabe eines Befehls einsetzen (statt veralteter Backticks `` `befehl` ``) |
| `$((1 + 2))` | Rechnen (nur ganze Zahlen) |

### Sonderparameter

| Parameter | Bedeutung |
| --- | --- |
| `$0` | Name des Skripts |
| `$1` … `$9` | Positionsparameter |
| `$#` | Anzahl der Parameter |
| `"$@"` | Alle Parameter, korrekt einzeln gequotet |
| `$?` | Exit-Code des letzten Befehls |
| `$$` | PID des Skripts |
| `$!` | PID des letzten Hintergrundprozesses |

---

## Bedingungen

```bash
if [[ -f "$file" && -r "$file" ]]; then
    echo "file exists and is readable"
elif [[ -d "$file" ]]; then
    echo "it is a directory"
else
    echo "not found"
fi
```

| Test | Wahr, wenn … |
| --- | --- |
| `-f datei` / `-d verz` / `-e pfad` | Datei / Verzeichnis / irgendwas existiert |
| `-r` / `-w` / `-x` | lesbar / schreibbar / ausführbar |
| `-s datei` | Datei nicht leer |
| `-z "$s"` / `-n "$s"` | String leer / nicht leer |
| `"$a" == "$b"` | Strings gleich (in `[[ ]]` auch Muster: `== *.log`) |
| `"$s" =~ ^[0-9]+$` | Regex-Treffer |
| `$a -eq $b`, `-ne`, `-lt`, `-le`, `-gt`, `-ge` | Zahlenvergleich |
| `(( a > b ))` | Zahlenvergleich in arithmetischer Schreibweise |

`[[ … ]]` ist die Bash-Variante und robuster als `[ … ]` (kein Word Splitting, `&&`/`||` erlaubt).

```bash
case "$1" in
    start|on)   echo "switching on" ;;
    stop|off)   echo "switching off" ;;
    status)     echo "querying" ;;
    *)          echo "usage: $0 {on|off|status}"; exit 1 ;;
esac
```

---

## Schleifen

```bash
# over a list
for host in pi-mqtt pi-beamer pi-camera; do
    ssh "$host" uptime
done

# over files (no 'ls' parsing!)
for f in /var/log/lab/*.log; do
    [[ -e "$f" ]] || continue          # no match -> pattern stays literal
    gzip "$f"
done

# counting
for i in {1..5}; do echo "try $i"; done

# reading a file line by line
while IFS= read -r line; do
    echo "host: $line"
done < hosts.txt

# retry loop
for attempt in {1..10}; do
    nc -z broker.lab.local 8883 && break
    sleep 3
done
```

---

## Funktionen

```bash
is_reachable() {
    local host="$1"
    ping -c1 -W1 "$host" >/dev/null 2>&1      # exit code is the return value
}

if is_reachable 192.168.10.21; then
    echo "online"
fi
```

* Variablen in Funktionen mit `local` deklarieren.
* `return` liefert nur einen **Exit-Code** (0–255); Daten gibt man per `echo` aus und fängt sie mit `$(…)` ab.

---

## Ein- und Ausgabe umleiten

| Schreibweise | Bedeutung |
| --- | --- |
| `> datei` | stdout in Datei (überschreiben) |
| `>> datei` | stdout anhängen |
| `2> datei` | stderr in Datei |
| `> datei 2>&1` / `&> datei` | stdout **und** stderr |
| `< datei` | Datei als stdin |
| `a \| b` | stdout von `a` als stdin von `b` |
| `> /dev/null` | Ausgabe verwerfen |
| `<<EOF … EOF` | Heredoc (→ [Heredoc](../Code-Formatierung/Heredoc.md)) |

Meldungen und Fehler gehören nach **stderr** (`>&2`), Nutzdaten nach **stdout** – dann lässt sich das Skript in Pipes verwenden.

---

## Exit-Codes

Jeder Befehl liefert einen **Exit-Code**: `0` = Erfolg, alles andere = Fehler. Eigene Skripte sollten das ebenfalls tun – dann funktionieren sie mit `&&`, `||`, `if`, Cron, systemd und Ansible.

```bash
command && echo "ok" || echo "failed"
exit 0     # success
exit 1     # general error
exit 2     # wrong usage (convention)
```

---

## Aufräumen mit trap

```bash
tmpdir="$(mktemp -d)"
cleanup() { rm -rf "$tmpdir"; }
trap cleanup EXIT              # runs on normal end, error (set -e) and Ctrl+C

# work with "$tmpdir" ...
```

**Mehrfachstart verhindern** (z. B. bei Cron-Jobs):

```bash
exec 9>/var/lock/backup.lock
flock -n 9 || { echo "already running" >&2; exit 1; }
```

---

## Typische Fallen

| Falle | Problem | Richtig |
| --- | --- | --- |
| `a = 5` | Leerzeichen um `=` → Befehl `a` wird gesucht | `a=5` |
| `cd $dir; rm -rf *` | Schlägt `cd` fehl, wird im **falschen Verzeichnis** gelöscht | `cd "$dir" \|\| exit 1` bzw. `set -e` |
| `for f in $(ls *.txt)` | Bricht bei Leerzeichen in Dateinamen | `for f in *.txt` |
| `cat file \| while read …; do count=…` | Pipe startet Subshell, Variable ist danach weg | `while …; done < file` |
| Windows-Zeilenenden | `bad interpreter: /bin/bash^M` | `dos2unix`, `.gitattributes` (→ [Cross-Plattform-Tricks](../Workarounds%20%26%20Hacks/Cross-Plattform-Tricks.md)) |
| Passwort im Skript | Landet in Git und in der Prozessliste | Umgebungsvariable, Datei mit Rechten `600` |
| `sudo echo x > /etc/datei` | Die Umleitung macht die **eigene** Shell, nicht sudo | `echo x \| sudo tee /etc/datei` |
| `~` in Anführungszeichen | `"~/datei"` wird **nicht** expandiert | `"$HOME/datei"` |

---

## Werkzeuge

* **[ShellCheck](https://www.shellcheck.net/)** findet die meisten Fehler automatisch: `sudo apt install shellcheck`, dann `shellcheck skript.sh`. Gibt es auch als Erweiterung für VS Code.
* **shfmt** formatiert Skripte einheitlich.
* `bash -n skript.sh` prüft nur die Syntax, ohne auszuführen.
* `bash -x skript.sh` zeigt jeden ausgeführten Befehl.

---

## Wann lieber Python?

Bash ist ideal, um **Befehle zu verketten**. Sobald ein Skript

* länger als etwa **100 Zeilen** wird,
* **JSON**, Datenstrukturen oder Fließkommazahlen verarbeitet,
* HTTP-APIs oder MQTT anspricht,
* oder echte **Fehlerbehandlung** braucht,

ist **Python** meist die bessere Wahl – lesbarer, testbar und plattformunabhängig.

---

## Checkliste

* [ ] Shebang `#!/usr/bin/env bash`
* [ ] `set -euo pipefail`
* [ ] Kopfkommentar: Zweck, Aufruf, Parameter
* [ ] Alle Variablen gequotet
* [ ] Parameter geprüft, `usage`-Meldung
* [ ] Meldungen nach stderr, sinnvolle Exit-Codes
* [ ] Temporäre Dateien mit `mktemp` + `trap` aufräumen
* [ ] Keine Passwörter im Skript
* [ ] `shellcheck` ohne Warnungen
* [ ] Ausführbar (`chmod +x`) und im Repository versioniert

---
