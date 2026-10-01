# 🧭 Backups nach Domäne

„Mach mal ein Backup“ bedeutet bei einer VM etwas völlig anderes als bei einer Datenbank oder einem Docker-Container. Dieses Dokument zeigt für die wichtigsten Domänen, **was** gesichert werden muss und **wie** man es **konsistent** sichert.

<!-- TOC -->
## Inhaltsverzeichnis

- [Grundfrage: Was muss eigentlich gesichert werden?](#grundfrage-was-muss-eigentlich-gesichert-werden)
- [Virtuelle Maschinen](#virtuelle-maschinen)
- [Container](#container)
  - [Docker](#docker)
  - [LXC (Proxmox-Container)](#lxc-proxmox-container)
  - [Kubernetes](#kubernetes)
- [Datenbanken](#datenbanken)
- [Dateien und Konfiguration](#dateien-und-konfiguration)
- [openHAB](#openhab)
- [Netzwerkgeräte und Embedded-Systeme](#netzwerkgeräte-und-embedded-systeme)
- [Übersicht](#übersicht)
<!-- /TOC -->

## Grundfrage: Was muss eigentlich gesichert werden?

Man unterscheidet drei Arten von Daten:

| Art | Beispiel | Sichern? |
| --- | --- | --- |
| **Reproduzierbar** | Betriebssystem, installierte Pakete, Docker-Images, kompilierter Code | Nicht zwingend – lässt sich neu erzeugen (z. B. per Ansible, Dockerfile) |
| **Konfiguration** | `/etc`, openHAB-Konfiguration, `docker-compose.yml`, Ansible-Repo | **Ja**, idealerweise versioniert in Git |
| **Zustand / Nutzdaten** | Datenbanken, Messdaten, Uploads, Volumes, Persistenz | **Ja**, regelmäßig und konsistent |

Wer reproduzierbare Teile weglässt, spart Platz und Zeit. Wer dagegen nur „das ganze System“ als Image sichert, kann zwar schnell wiederherstellen, weiß aber oft nicht, was eigentlich darin steckt.

---

## Virtuelle Maschinen

Bei einer VM sichert man die **virtuelle Festplatte + die VM-Konfiguration** (CPU, RAM, Netzwerk).

| Verfahren | Beschreibung |
| --- | --- |
| **Snapshot** | Schneller Zustand vor Änderungen, liegt auf demselben Storage – **kein Backup** (siehe [Backup-Arten](Backup-Arten.md#snapshot)) |
| **Vollbackup der VM** | Komplettes Abbild auf anderem Speicher, z. B. Proxmox `vzdump` |
| **Inkrementell mit Dirty Bitmaps** | Der Hypervisor merkt sich geänderte Blöcke, nur diese werden gesichert (Proxmox Backup Server) |
| **Backup im Gast** | Dateien/Datenbanken innerhalb der VM klassisch sichern |

**Proxmox (`vzdump`) – Modi:**

| Modus | Ablauf | Konsistenz | Ausfallzeit |
| --- | --- | --- | --- |
| `snapshot` | VM läuft weiter, Backup aus Live-Zustand | mit **QEMU Guest Agent** dateisystemkonsistent (`fsfreeze`) | keine |
| `suspend` | VM wird kurz pausiert | gut | kurz |
| `stop` | VM wird heruntergefahren, gesichert, wieder gestartet | am höchsten | Dauer des Backups |

```bash
vzdump 101 --mode snapshot --compress zstd --storage nas-backup --notes-template '{{guestname}}'
```

> Für `snapshot` im Gast den **QEMU Guest Agent** installieren und in der VM-Konfiguration aktivieren. Sonst ist das Backup nur „absturzkonsistent“ – so, als hätte man den Stecker gezogen.

---

## Container

### Docker

Ein Docker-Container ist **wegwerfbar**. Das Image ist reproduzierbar (Dockerfile, Registry). Gesichert werden:

1. **Volumes und Bind-Mounts** (der Zustand)
2. **`docker-compose.yml`, `.env`, eigene Dockerfiles** (die Konfiguration, am besten in Git)
3. Bei eigenen Images ohne Registry ggf. das **Image** selbst (`docker save`)

```bash
# Ein benanntes Volume als tar sichern
docker run --rm \
  -v openhab_userdata:/data:ro \
  -v "$PWD":/backup \
  alpine tar czf /backup/openhab_userdata_$(date +%F).tar.gz -C /data .

# Wiederherstellen
docker run --rm \
  -v openhab_userdata:/data \
  -v "$PWD":/backup \
  alpine sh -c "cd /data && tar xzf /backup/openhab_userdata_2026-10-01.tar.gz"
```

> Laufende Datenbank-Container **nicht** über ihr Volume sichern, sondern über einen Dump (siehe unten), oder den Container vorher stoppen.

**`docker commit` ist kein Backup-Verfahren** – es erzeugt ein undurchsichtiges Image, dessen Entstehung niemand nachvollziehen kann.

### LXC (Proxmox-Container)

LXC-Container sind „Systemcontainer“ mit eigenem Init-System – sie verhalten sich eher wie eine leichte VM. Gesichert werden sie wie VMs mit `vzdump` (Modi `snapshot`, `suspend`, `stop`).

### Kubernetes

Gesichert werden die **Manifeste** (in Git, GitOps), die **Persistent Volumes** und ggf. der **etcd**-Zustand des Clusters. Werkzeug: Velero.

---

## Datenbanken

Die wichtigste Regel: **Die Datendateien einer laufenden Datenbank niemals einfach kopieren.** Während der Kopie schreibt die Datenbank weiter, das Ergebnis ist inkonsistent und oft nicht mehr startbar.

Stattdessen den **Dump** bzw. das Backup-Werkzeug der Datenbank verwenden:

| Datenbank | Backup | Restore |
| --- | --- | --- |
| **MariaDB / MySQL** | `mysqldump --single-transaction --routines --all-databases > dump.sql` | `mysql < dump.sql` |
| **PostgreSQL** | `pg_dump -Fc smarthome > smarthome.dump` / `pg_dumpall > all.sql` | `pg_restore -d smarthome smarthome.dump` |
| **SQLite** | `sqlite3 data.db ".backup 'data_backup.db'"` | Datei zurückkopieren |
| **InfluxDB 2.x** | `influx backup /backup/influx_$(date +%F)` | `influx restore /backup/...` |
| **MongoDB** | `mongodump --out /backup/mongo` | `mongorestore /backup/mongo` |
| **Redis** | `redis-cli BGSAVE`, dann `dump.rdb` kopieren | `dump.rdb` zurückkopieren |

Alternativen für große Datenbanken: physische Backups (`mariabackup`, `pg_basebackup`) und **Point-in-Time-Recovery** über Transaktionslogs (Binlog, WAL) – damit lässt sich ein beliebiger Zeitpunkt wiederherstellen, z. B. „eine Minute vor dem versehentlichen `DELETE`“.

---

## Dateien und Konfiguration

| Was | Wie |
| --- | --- |
| `/etc`, Konfigurationsdateien | [Git](Git%20als%20Backup.md) (`etckeeper`) **und** `tar` |
| Home-Verzeichnisse, Projektordner | `rsync`, `tar`, BorgBackup, restic |
| Große Datenmengen (Messdaten, Videos) | `rsync` auf NAS, deduplizierende Tools |
| Code | Git-Remote (GitHub/GitLab) – zusätzlich gelegentlich `git bundle` oder Mirror |

---

## openHAB

openHAB bringt ein eigenes Backup-Werkzeug mit, das Konfiguration (`conf`) und Benutzerdaten (`userdata`: Things, Items aus der UI, JSON-DB, Persistenz) zusammen sichert:

```bash
sudo openhab-cli backup /var/backups/openhab/openhab_$(date +%F).zip
sudo openhab-cli restore /var/backups/openhab/openhab_2026-10-01.zip   # openHAB vorher stoppen
```

Zusätzlich sinnvoll:

* `conf/` (Textkonfiguration: `.items`, `.things`, `.rules`, Skripte) in **Git**.
* Persistenz-Datenbank (z. B. InfluxDB, MariaDB) über deren eigenes Backup.

---

## Netzwerkgeräte und Embedded-Systeme

| Gerät | Backup |
| --- | --- |
| Router / FRITZ!Box | Konfiguration exportieren (mit Kennwort verschlüsselt) |
| OpenWrt | `sysupgrade -b /tmp/backup-$(hostname)-$(date +%F).tar.gz` |
| Managed Switch | `show running-config` bzw. Konfigurations-Export, z. B. per Ansible (`*_config` Module) |
| Raspberry Pi | SD-Karten-Image (selten) **plus** Konfiguration in Ansible/Git (häufig) |
| Mikrocontroller (ESP32, Arduino) | Quellcode + Konfiguration in Git; die Firmware ist reproduzierbar |

---

## Übersicht

| Domäne | Was sichern | Werkzeug | Konsistenz beachten |
| --- | --- | --- | --- |
| VM | Disk + Config | `vzdump`, PBS | Guest Agent, `snapshot`/`stop` |
| Docker | Volumes, Compose-Dateien | `tar` über Hilfscontainer, Git | DB-Container per Dump |
| LXC | Container-Dateisystem + Config | `vzdump` | wie VM |
| Datenbank | Dump bzw. physisches Backup | `mysqldump`, `pg_dump`, `influx backup` | nie Rohdateien im Betrieb |
| Konfiguration | Textdateien | Git, `etckeeper`, Ansible | – |
| openHAB | conf + userdata | `openhab-cli backup` | openHAB beim Restore stoppen |
| Netzwerk | Konfiguration | Export, Ansible | – |

---
