# ⏱️ Backup-Skripte & Intervalle

Backups müssen **automatisch** laufen. Unter Linux geschieht das klassisch mit **Cron**: Ein Skript wird in festen Intervallen ausgeführt – z. B. wöchentlich, monatlich und jährlich. Grundlagen zu Cron (Syntax, Fallstricke, systemd-Timer): [Cron & systemd-Timer](../Linux%20&%20Werkzeuge/Cron%20&%20systemd-Timer.md).

<!-- TOC -->
## Inhaltsverzeichnis

- [Intervalle wählen](#intervalle-wählen)
- [Aufbau eines robusten Backup-Skripts](#aufbau-eines-robusten-backup-skripts)
- [Gemeinsame Funktionen](#gemeinsame-funktionen)
- [Wöchentliches Skript](#wöchentliches-skript)
- [Monatliches Skript](#monatliches-skript)
- [Jährliches Skript](#jährliches-skript)
- [Crontab](#crontab)
- [Logs und Benachrichtigung](#logs-und-benachrichtigung)
<!-- /TOC -->

## Intervalle wählen

Das Intervall ergibt sich aus der Frage: **Wie viel Arbeit darf ich höchstens verlieren?** (RPO, siehe [Backup-Arten](Backup-Arten.md#rpo-und-rto)).

| Intervall | Typischer Inhalt | Aufbewahrung (Beispiel) |
| --- | --- | --- |
| stündlich | Datenbanken mit vielen Änderungen, Git-Auto-Commit | 24–48 Stück |
| täglich | Datenbanken, Nutzdaten, openHAB | 7–14 Stück |
| **wöchentlich** | Konfigurationen, Volumes, Dumps | **5 Stück** |
| **monatlich** | Vollbackups von VMs und Daten | **12 Stück** |
| **jährlich** | Archiv zum Jahresende (Nachweis, Semesterabschluss) | **5 Jahre** oder dauerhaft |

Intervalle werden **zeitlich versetzt**, damit nicht alle Jobs gleichzeitig laufen und sich nicht gegenseitig ausbremsen.

---

## Aufbau eines robusten Backup-Skripts

Jedes Backup-Skript sollte:

1. **bei Fehlern abbrechen** (`set -euo pipefail`),
2. **nicht doppelt laufen** (Lock mit `flock`),
3. **prüfen, ob das Ziel verfügbar ist** (NAS gemountet?),
4. **in eine temporäre Datei schreiben und erst am Ende umbenennen** (keine halben Backups mit gültigem Namen),
5. **das Ergebnis prüfen** (Archiv lesbar, Prüfsumme),
6. **alte Backups löschen** ([Retention](Retention%20&%20Rotation.md)),
7. **protokollieren** und bei Fehlern **benachrichtigen**,
8. mit **Exit-Code ≠ 0** enden, wenn etwas schiefging.

---

## Gemeinsame Funktionen

Damit die drei Skripte nicht dreimal denselben Code enthalten, liegen die gemeinsamen Teile in einer Bibliothek (siehe [Template Method](../Design%20Pattern/Verhaltensmuster/Template%20Method.md): Ablauf fest, Details variabel).

`/usr/local/lib/backup/common.sh`:

```bash
#!/usr/bin/env bash
# Shared functions for backup scripts.
set -euo pipefail

BACKUP_ROOT="/mnt/nas/backups/$(hostname)"
LOG_TAG="backup"

log()  { echo "$(date '+%F %T') [$LOG_TAG] $*"; logger -t "$LOG_TAG" -- "$*"; }
fail() { log "ERROR: $*"; exit 1; }

require_mount() {
  # Verhindert, dass bei fehlendem NAS in den leeren Mountpoint (lokale Platte!) geschrieben wird.
  mountpoint -q /mnt/nas || fail "/mnt/nas is not mounted"
}

# create_archive <target-file> <path>...
create_archive() {
  local target=$1; shift
  local tmp="${target}.part"
  tar --create --zstd --file "$tmp" \
      --exclude='*.log' --exclude='cache' --exclude='tmp' \
      "$@"
  tar --list --zstd --file "$tmp" > /dev/null || fail "archive $tmp is not readable"
  mv "$tmp" "$target"
  sha256sum "$target" > "${target}.sha256"
  log "created $(basename "$target") ($(du -h "$target" | cut -f1))"
}

# keep_last <dir> <pattern> <count>
keep_last() {
  local dir=$1 pattern=$2 keep=$3
  find "$dir" -maxdepth 1 -name "$pattern" -printf '%T@ %p\n' \
    | sort -rn | tail -n +$((keep + 1)) | cut -d' ' -f2- \
    | while read -r old; do
        rm -f -- "$old" "${old}.sha256"
        log "removed old backup $(basename "$old")"
      done
}
```

---

## Wöchentliches Skript

Sichert Konfigurationen und Datenbank-Dumps, behält die **letzten 5**.

`/usr/local/bin/backup_weekly.sh`:

```bash
#!/usr/bin/env bash
# Weekly backup: configuration + database dump, keep last 5.
source /usr/local/lib/backup/common.sh
LOG_TAG="backup-weekly"

exec 9>/run/lock/backup.lock
flock -n 9 || fail "another backup is running"

require_mount
DEST="$BACKUP_ROOT/weekly"
mkdir -p "$DEST"
STAMP=$(date +%G-W%V)          # ISO-Kalenderwoche, z. B. 2026-W40

# Datenbank konsistent dumpen (nie die Rohdateien kopieren!)
DUMP_DIR=$(mktemp -d)
trap 'rm -rf "$DUMP_DIR"' EXIT
mysqldump --single-transaction --routines --all-databases > "$DUMP_DIR/mariadb.sql"

create_archive "$DEST/weekly_${STAMP}.tar.zst" /etc /opt/scripts "$DUMP_DIR"
keep_last "$DEST" 'weekly_*.tar.zst' 5
log "weekly backup finished"
```

---

## Monatliches Skript

Vollbackup inkl. Nutzdaten, behält die **letzten 12**.

`/usr/local/bin/backup_monthly.sh`:

```bash
#!/usr/bin/env bash
# Monthly full backup of configuration and data, keep last 12.
source /usr/local/lib/backup/common.sh
LOG_TAG="backup-monthly"

exec 9>/run/lock/backup.lock
flock -w 3600 9 || fail "could not get lock within 1 h"

require_mount
DEST="$BACKUP_ROOT/monthly"
mkdir -p "$DEST"
STAMP=$(date +%Y-%m)

openhab-cli backup "/tmp/openhab_${STAMP}.zip" >/dev/null
create_archive "$DEST/monthly_${STAMP}.tar.zst" /etc /opt /srv/data "/tmp/openhab_${STAMP}.zip"
rm -f "/tmp/openhab_${STAMP}.zip"

keep_last "$DEST" 'monthly_*.tar.zst' 12
log "monthly backup finished"
```

---

## Jährliches Skript

Archiv zum Jahreswechsel. Behält die **letzten 5 Jahre** und legt zusätzlich eine Kopie auf ein zweites Ziel (offsite).

`/usr/local/bin/backup_yearly.sh`:

```bash
#!/usr/bin/env bash
# Yearly archive, keep last 5, copy offsite.
source /usr/local/lib/backup/common.sh
LOG_TAG="backup-yearly"

exec 9>/run/lock/backup.lock
flock -w 7200 9 || fail "could not get lock within 2 h"

require_mount
DEST="$BACKUP_ROOT/yearly"
mkdir -p "$DEST"
YEAR=$(date -d 'yesterday' +%Y)   # läuft am 1. Januar → sichert das abgelaufene Jahr

create_archive "$DEST/yearly_${YEAR}.tar.zst" /etc /opt /srv/data

# Offsite-Kopie (z. B. zweites NAS in anderem Gebäude)
rsync -a --partial "$DEST/yearly_${YEAR}.tar.zst"* backup@offsite-nas:/backups/$(hostname)/

keep_last "$DEST" 'yearly_*.tar.zst' 5
log "yearly backup finished"
```

---

## Crontab

`sudo crontab -e` (oder als Datei `/etc/cron.d/backup` mit zusätzlicher Benutzerspalte):

```cron
# m  h  dom mon dow  command
# Wöchentlich: Sonntag 02:30
30  2  *   *   0    /usr/local/bin/backup_weekly.sh  >> /var/log/backup_weekly.log  2>&1
# Monatlich: am 1. um 03:30
30  3  1   *   *    /usr/local/bin/backup_monthly.sh >> /var/log/backup_monthly.log 2>&1
# Jährlich: am 1. Januar um 04:30
30  4  1   1   *    /usr/local/bin/backup_yearly.sh  >> /var/log/backup_yearly.log  2>&1
```

Alternativ gibt es die Kurzformen `@weekly`, `@monthly`, `@yearly` – diese laufen aber alle um **00:00 Uhr** und damit potenziell gleichzeitig. Mit festen, versetzten Zeiten behält man die Kontrolle.

Der gemeinsame Lock (`/run/lock/backup.lock`) sorgt dafür, dass z. B. am 1. Januar, der auf einen Sonntag fällt, die Skripte **nacheinander** statt gleichzeitig laufen.

---

## Logs und Benachrichtigung

* Logs nach `/var/log/backup_*.log` (mit `logrotate` begrenzen) und über `logger` ins Journal: `journalctl -t backup-weekly`
* **Benachrichtigung bei Fehlern**, sonst merkt es niemand: Mail über `MAILTO=` in der Crontab, ein Push-Dienst (z. B. ntfy, Gotify), eine MQTT-Nachricht an openHAB oder ein Healthcheck-Dienst, der Alarm schlägt, wenn sich ein Job **nicht** gemeldet hat:

```bash
# am Ende des Skripts, nur bei Erfolg erreicht:
curl -fsS -m 10 --retry 3 https://healthchecks.example.org/ping/<uuid> > /dev/null
```

Ein Backup, das seit Wochen still fehlschlägt, ist schlimmer als keines – weil man sich in falscher Sicherheit wiegt.

---
