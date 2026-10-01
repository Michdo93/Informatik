# 📦 Tar & NAS

Die einfachste und robusteste Form eines Backups: Daten mit `tar` in **eine Archivdatei** packen und regelmäßig auf ein **NAS** schieben. Keine spezielle Software, lesbar auf jedem Linux-System – auch noch in zwanzig Jahren.

<!-- TOC -->
## Inhaltsverzeichnis

- [Warum tar?](#warum-tar)
- [Die wichtigsten tar-Befehle](#die-wichtigsten-tar-befehle)
- [Prüfsummen](#prüfsummen)
- [Inkrementell mit tar](#inkrementell-mit-tar)
- [NAS einbinden](#nas-einbinden)
  - [NFS (Linux ↔ Linux/NAS)](#nfs-linux--linuxnas)
  - [SMB/CIFS (Windows-Freigabe, Synology, FRITZ!Box)](#smbcifs-windows-freigabe-synology-fritzbox)
  - [Die gefährlichste Falle](#die-gefährlichste-falle)
- [Übertragen mit rsync](#übertragen-mit-rsync)
- [Wann lieber ein spezialisiertes Werkzeug?](#wann-lieber-ein-spezialisiertes-werkzeug)
<!-- /TOC -->

## Warum tar?

`tar` (*tape archive*, ursprünglich für Magnetbänder) fasst viele Dateien zu einer zusammen und **erhält dabei Rechte, Besitzer, Zeitstempel und symbolische Links**. Eine Kopie mit `cp` oder über eine Windows-Freigabe verliert diese Informationen oft.

| Endung | Kompression | Hinweis |
| --- | --- | --- |
| `.tar` | keine | schnell, groß |
| `.tar.gz` / `.tgz` | gzip | überall verfügbar |
| `.tar.xz` | xz | sehr klein, langsam |
| `.tar.zst` | zstd | **Empfehlung**: schnell und klein, mehrkernfähig |

---

## Die wichtigsten tar-Befehle

```bash
# Erstellen (c = create, f = file)
tar -czf backup.tar.gz /etc /opt/scripts          # gzip
tar --zstd -cf backup.tar.zst /etc /opt/scripts   # zstd

# Mit Ausschlüssen
tar --zstd -cf backup.tar.zst \
    --exclude='*.log' --exclude='node_modules' --exclude='.venv' \
    /home/projekt

# Inhalt auflisten (t = list) – prüft zugleich, ob das Archiv lesbar ist
tar --zstd -tf backup.tar.zst

# Entpacken (x = extract) in ein Zielverzeichnis
tar --zstd -xf backup.tar.zst -C /tmp/restore

# Nur eine Datei wiederherstellen
tar --zstd -xf backup.tar.zst -C /tmp/restore etc/mosquitto/mosquitto.conf
```

> `tar` entfernt den führenden `/` aus Pfaden („Removing leading `/` from member names“). Das ist gewollt: Beim Entpacken landet nichts versehentlich direkt in `/etc`, sondern im Zielverzeichnis.

**Dateinamen mit Datum und Host:**

```bash
backup_$(hostname)_$(date +%F_%H%M).tar.zst   # backup_pi-beamer_2026-10-01_0230.tar.zst
```

Datum im Format `YYYY-MM-DD` (ISO 8601) – so sortieren sich die Dateien automatisch chronologisch.

---

## Prüfsummen

```bash
sha256sum backup.tar.zst > backup.tar.zst.sha256
sha256sum -c backup.tar.zst.sha256      # später prüfen: "backup.tar.zst: OK"
```

So erkennt man, ob ein Archiv bei der Übertragung oder durch defekte Hardware beschädigt wurde („Bit Rot“).

---

## Inkrementell mit tar

GNU tar kann selbst inkrementelle Backups erstellen, über eine **Snapshot-Datei** (`.snar`), in der es sich den Stand merkt:

```bash
# Vollbackup (neue Snapshot-Datei)
rm -f data.snar
tar --zstd -cf full.tar.zst --listed-incremental=data.snar /srv/data

# Inkrementell (nur Änderungen seit dem letzten Lauf mit dieser .snar)
tar --zstd -cf inc1.tar.zst --listed-incremental=data.snar /srv/data

# Wiederherstellung: Voll + alle Inkremente in Reihenfolge
tar --zstd -xf full.tar.zst --listed-incremental=/dev/null -C /
tar --zstd -xf inc1.tar.zst --listed-incremental=/dev/null -C /
```

Ein vollständiges Skript mit Monats-Voll + Wochen-Inkrement: [Retention & Rotation](Retention%20&%20Rotation.md#monatlich-voll-wöchentlich-inkrementell).

---

## NAS einbinden

### NFS (Linux ↔ Linux/NAS)

```bash
sudo apt install nfs-common
sudo mkdir -p /mnt/nas
```

`/etc/fstab`:

```text
nas:/volume1/backups  /mnt/nas  nfs  defaults,_netdev,nofail,x-systemd.automount  0  0
```

### SMB/CIFS (Windows-Freigabe, Synology, FRITZ!Box)

```bash
sudo apt install cifs-utils
sudo install -m 600 /dev/null /root/.smb-nas
printf 'username=backup\npassword=...\n' | sudo tee /root/.smb-nas > /dev/null
```

`/etc/fstab`:

```text
//nas/backups  /mnt/nas  cifs  credentials=/root/.smb-nas,uid=0,gid=0,_netdev,nofail,x-systemd.automount  0  0
```

Optionen erklärt:

| Option | Bedeutung |
| --- | --- |
| `_netdev` | Netzwerkgerät – erst einhängen, wenn das Netz da ist |
| `nofail` | System bootet auch, wenn das NAS nicht erreichbar ist |
| `x-systemd.automount` | Wird erst beim ersten Zugriff eingehängt |
| `credentials=` | Zugangsdaten aus Datei (Rechte `600`), nicht in der `fstab` |

### Die gefährlichste Falle

Ist das NAS **nicht** eingehängt, ist `/mnt/nas` ein ganz normales, leeres Verzeichnis auf der lokalen Platte. Das Backup-Skript schreibt dann fröhlich dorthin – bis die Systemplatte voll ist. **Deshalb immer prüfen:**

```bash
mountpoint -q /mnt/nas || { echo "NAS not mounted" >&2; exit 1; }
```

---

## Übertragen mit rsync

Wenn das Archiv lokal erstellt und dann übertragen werden soll – oder ein Verzeichnis direkt gespiegelt werden soll:

```bash
# Archiv auf das NAS kopieren (über SSH, mit Fortsetzung bei Abbruch)
rsync -av --partial backup_*.tar.zst* backup@nas:/volume1/backups/$(hostname)/

# Verzeichnis spiegeln
rsync -aHAX --info=progress2 /srv/data/ /mnt/nas/data-mirror/
```

| Option | Bedeutung |
| --- | --- |
| `-a` | Archivmodus: rekursiv, Rechte, Zeiten, Links |
| `-H -A -X` | Hardlinks, ACLs, erweiterte Attribute |
| `--partial` | Abgebrochene Übertragungen fortsetzen |
| `--delete` | Im Ziel löschen, was in der Quelle fehlt – **Vorsicht**, das macht aus dem Backup einen Spiegel (siehe [Backup-Arten](Backup-Arten.md#synchronisation-und-spiegelung-sind-kein-backup)) |
| `-n` / `--dry-run` | Nur anzeigen, was passieren würde |

> Achtung beim abschließenden `/`: `rsync quelle/ ziel/` kopiert den **Inhalt** von `quelle`, `rsync quelle ziel/` kopiert den **Ordner** `quelle` nach `ziel/quelle`.

**Versionierte Spiegel mit Hardlinks:** Mit `--link-dest` werden unveränderte Dateien als Hardlink auf das vorherige Backup angelegt. Jeder Ordner sieht aus wie ein Vollbackup, belegt aber nur den Platz der Änderungen (Prinzip von *rsnapshot* und Apples Time Machine).

```bash
rsync -a --delete --link-dest=/mnt/nas/data/2026-09-30 /srv/data/ /mnt/nas/data/2026-10-01/
```

---

## Wann lieber ein spezialisiertes Werkzeug?

Für große Datenmengen, viele Versionen oder verschlüsselte Offsite-Backups sind deduplizierende Werkzeuge effizienter:

| Werkzeug | Besonderheit |
| --- | --- |
| **BorgBackup** | Deduplizierung, Kompression, Verschlüsselung, eingebaute Retention (`borg prune`) |
| **restic** | Wie Borg, zusätzlich direkt in S3, Backblaze, SFTP |
| **Proxmox Backup Server** | Für Proxmox-VMs und -Container, inkrementell + dedupliziert |
| **rsnapshot** | Versionierte rsync-Spiegel mit Hardlinks |

`tar` bleibt trotzdem die Grundlage, die man verstehen und im Notfall ohne weitere Software anwenden können muss.

---
