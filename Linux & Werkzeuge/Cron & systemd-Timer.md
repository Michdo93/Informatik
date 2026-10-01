# ⏰ Cron & systemd-Timer

Wiederkehrende Aufgaben – Backups, Updates, Aufräumen, Messwerte abholen – sollen automatisch laufen. Unter Linux gibt es dafür zwei Werkzeuge: den Klassiker **Cron** und die moderne Alternative **systemd-Timer**.

<!-- TOC -->
## Inhaltsverzeichnis

- [Begriffe: Cron, Crontab, Cron-Job, Cron-Task](#begriffe-cron-crontab-cron-job-cron-task)
- [Crontab-Syntax](#crontab-syntax)
  - [Beispiele](#beispiele)
  - [Kurzschreibweisen](#kurzschreibweisen)
- [Crontab bearbeiten](#crontab-bearbeiten)
- [Systemweite Cron-Verzeichnisse](#systemweite-cron-verzeichnisse)
- [Typische Fallen](#typische-fallen)
- [anacron](#anacron)
- [systemd-Timer](#systemd-timer)
- [Cron oder systemd-Timer?](#cron-oder-systemd-timer)
- [Mit Ansible verwalten](#mit-ansible-verwalten)
<!-- /TOC -->

## Begriffe: Cron, Crontab, Cron-Job, Cron-Task

| Begriff | Bedeutung |
| --- | --- |
| **cron** | Der **Dienst** (Daemon `cron`/`crond`), der jede Minute prüft, ob etwas auszuführen ist |
| **crontab** | Die **Tabelle** mit den Einträgen (*cron table*) – und der Befehl, um sie zu bearbeiten |
| **Cron-Job** | **Ein Eintrag** in der Crontab: Zeitplan + Befehl |
| **Cron-Task** | Umgangssprachlich meist dasselbe wie Cron-Job; manchmal ist mit *Task* die **auszuführende Aufgabe** (das Skript) gemeint und mit *Job* die **geplante Ausführung** davon. In Ansible heißt das Modul `cron`, jeder Eintrag ist ein *Job*. |

**Namensherkunft:** von griechisch **χρόνος (chronos) = Zeit**. Cron erschien in den 1970er-Jahren in Unix (Version 7); die heute verbreitete Implementierung stammt von Paul Vixie (1987, „Vixie cron“).

---

## Crontab-Syntax

```text
┌───────────── Minute        (0–59)
│ ┌─────────── Stunde        (0–23)
│ │ ┌───────── Tag im Monat  (1–31)
│ │ │ ┌─────── Monat         (1–12 oder jan–dec)
│ │ │ │ ┌───── Wochentag     (0–7, 0 und 7 = Sonntag, oder sun–sat)
│ │ │ │ │
* * * * *  Befehl
```

| Zeichen | Bedeutung | Beispiel |
| --- | --- | --- |
| `*` | jeder Wert | `* * * * *` = jede Minute |
| `,` | Liste | `0 8,12,18 * * *` = 8, 12 und 18 Uhr |
| `-` | Bereich | `0 8-17 * * 1-5` = stündlich 8–17 Uhr, Mo–Fr |
| `/` | Schrittweite | `*/15 * * * *` = alle 15 Minuten |

### Beispiele

| Zeitplan | Bedeutung |
| --- | --- |
| `30 2 * * *` | täglich um 02:30 |
| `0 3 * * 0` | **wöchentlich** sonntags um 03:00 |
| `0 4 1 * *` | **monatlich** am 1. um 04:00 |
| `0 5 1 1 *` | **jährlich** am 1. Januar um 05:00 |
| `*/5 * * * *` | alle 5 Minuten |
| `0 7 * * 1-5` | werktags um 07:00 |

### Kurzschreibweisen

| Makro | entspricht |
| --- | --- |
| `@reboot` | einmal nach dem Systemstart |
| `@hourly` | `0 * * * *` |
| `@daily` / `@midnight` | `0 0 * * *` |
| `@weekly` | `0 0 * * 0` |
| `@monthly` | `0 0 1 * *` |
| `@yearly` / `@annually` | `0 0 1 1 *` |

> **Tipp:** Zeitpläne mit **[crontab.guru](https://crontab.guru/)** prüfen.

---

## Crontab bearbeiten

```bash
crontab -e            # edit your own crontab
crontab -l            # list it
sudo crontab -e       # root's crontab
crontab -u pi -l      # crontab of user pi (as root)
```

Beispiel einer vollständigen Crontab:

```bash
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=""                                  # no mails; or an address

# m  h  dom mon dow  command
  0  3  *   *   0    /opt/backup/backup_weekly.sh  >> /var/log/backup/weekly.log  2>&1
  0  4  1   *   *    /opt/backup/backup_monthly.sh >> /var/log/backup/monthly.log 2>&1
  0  5  1   1   *    /opt/backup/backup_yearly.sh  >> /var/log/backup/yearly.log  2>&1
@reboot sleep 30 &&  /opt/lab/start_sensors.sh
```

Siehe [Backup-Skripte & Intervalle](../Backup-Strategien/Backup-Skripte%20%26%20Intervalle.md) für die zugehörigen Skripte.

---

## Systemweite Cron-Verzeichnisse

| Ort | Zweck |
| --- | --- |
| `/etc/crontab` | Systemcrontab, hat **zusätzliche Spalte für den Benutzer** |
| `/etc/cron.d/` | Einzelne Dateien im Format von `/etc/crontab` – ideal für Pakete und **Ansible** |
| `/etc/cron.hourly/`, `cron.daily/`, `cron.weekly/`, `cron.monthly/` | Ausführbare Skripte, die per `run-parts` im jeweiligen Rhythmus laufen |

```text
# /etc/cron.d/lab-backup  (note the user column!)
0 3 * * 0   root   /opt/backup/backup_weekly.sh >> /var/log/backup/weekly.log 2>&1
```

> ⚠️ Dateien in `/etc/cron.d/` und `cron.daily/` usw. dürfen **keine Punkte im Namen** haben (`backup.sh` wird von `run-parts` unter Debian **ignoriert**!) und müssen bei `cron.daily/` ausführbar sein.

---

## Typische Fallen

| Falle | Erklärung | Lösung |
| --- | --- | --- |
| **Anderer `PATH`** | Cron hat einen minimalen `PATH` (oft nur `/usr/bin:/bin`) | Absolute Pfade oder `PATH=` in der Crontab |
| **Anderes Arbeitsverzeichnis** | Cron startet im Home-Verzeichnis | Im Skript `cd "$(dirname "$0")"` |
| **Keine Umgebungsvariablen** | `.bashrc` wird nicht geladen | Variablen im Skript oder in der Crontab setzen |
| **`%` im Befehl** | `%` bedeutet in der Crontab **Zeilenumbruch** | Als `\%` escapen: `date +\%F` |
| **Tag im Monat UND Wochentag** | Sind **beide** gesetzt, gilt **ODER**: `0 3 1 * 1` = am 1. **und** jeden Montag | Im Skript prüfen: `[ "$(date +\%u)" = 1 ] && …` |
| **Keine Ausgabe sichtbar** | Ausgabe geht per Mail (oder verloren) | `>> logfile 2>&1` |
| **Rechner aus** | Verpasste Jobs werden **nicht** nachgeholt | `anacron` oder systemd-Timer mit `Persistent=true` |
| **Überlappung** | Job läuft länger als das Intervall | `flock -n /tmp/job.lock befehl` |
| **Zeitumstellung** | Jobs zwischen 2 und 3 Uhr laufen bei Sommer-/Winterzeit doppelt oder gar nicht | Kritische Jobs nicht in dieses Fenster legen |
| **Letzte Zeile** | Crontab ohne abschließenden Zeilenumbruch → letzte Zeile wird ignoriert | Immer mit Leerzeile enden |

**Debuggen:**

```bash
grep CRON /var/log/syslog          # Debian/Ubuntu/Raspberry Pi OS
journalctl -u cron --since today   # with systemd journal
```

---

## anacron

Für Rechner, die **nicht dauerhaft laufen** (Laptops, Laborrechner): **anacron** führt tägliche/wöchentliche/monatliche Jobs aus, **sobald der Rechner wieder an ist**, wenn sie verpasst wurden. Unter Debian/Ubuntu ist `anacron` für `cron.daily/weekly/monthly` meist bereits eingebunden.

---

## systemd-Timer

Moderne Alternative zu Cron. Ein Timer besteht aus **zwei Dateien**: einer `.service`-Unit (was?) und einer `.timer`-Unit (wann?).

```ini
# /etc/systemd/system/backup-weekly.service
[Unit]
Description=Weekly lab backup
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/opt/backup/backup_weekly.sh
User=root
Nice=10
IOSchedulingClass=idle
```

```ini
# /etc/systemd/system/backup-weekly.timer
[Unit]
Description=Run weekly lab backup

[Timer]
OnCalendar=Sun *-*-* 03:00:00
Persistent=true                 # catch up if the machine was off
RandomizedDelaySec=15min        # spread load across many machines

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup-weekly.timer
systemctl list-timers                       # all timers with next/last run
journalctl -u backup-weekly.service         # full log of every run
sudo systemctl start backup-weekly.service  # run manually now
systemd-analyze calendar "Sun *-*-* 03:00"  # check a schedule expression
```

**`OnCalendar`-Beispiele:** `daily`, `weekly`, `monthly`, `yearly`, `Mon..Fri 07:00`, `*-*-01 04:00` (monatlich), `*:0/15` (alle 15 Min.).

---

## Cron oder systemd-Timer?

| Kriterium | Cron | systemd-Timer |
| --- | --- | --- |
| Einrichtung | eine Zeile | zwei Dateien |
| Logging | selbst umleiten | automatisch im Journal |
| Verpasste Läufe nachholen | nur mit anacron | `Persistent=true` |
| Abhängigkeiten (Netzwerk, Mounts) | nein | `After=`, `Requires=` |
| Ressourcenbegrenzung | nein | `Nice=`, `CPUQuota=`, `MemoryMax=` |
| Überlappung verhindern | `flock` | automatisch (Unit läuft nur einmal) |
| Portabilität | jedes Unix, auch BusyBox | nur systemd-Systeme |
| Bekanntheit | sehr hoch | mittel |

**Empfehlung:** Für einfache Aufgaben ist Cron völlig in Ordnung. Für **Backups** und alles, was Netzwerk/NAS braucht oder nachgeholt werden muss, sind systemd-Timer robuster.

---

## Mit Ansible verwalten

Cron-Jobs **nicht per Hand** auf jedem Gerät eintragen, sondern über Ansible – dann sind sie dokumentiert und reproduzierbar (→ [Best Practice Ansible](../Best%20Practices/Ansible.md)):

```yaml
- name: Weekly backup job
  ansible.builtin.cron:
    name: "lab weekly backup"          # identifies the job - keeps it idempotent
    cron_file: lab-backup              # -> /etc/cron.d/lab-backup
    user: root
    weekday: "0"
    hour: "3"
    minute: "0"
    job: "/opt/backup/backup_weekly.sh >> /var/log/backup/weekly.log 2>&1"
```

---
