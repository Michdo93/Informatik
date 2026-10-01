# 🔄 Export & Import

Ein **Export** schreibt Daten oder Konfiguration in ein **portables, oft menschenlesbares Format**, das sich in ein anderes System (oder eine andere Version) wieder **importieren** lässt. Exporte sind kein Ersatz für Backups, aber eine wichtige Ergänzung.

<!-- TOC -->
## Inhaltsverzeichnis

- [Backup vs. Export](#backup-vs-export)
- [Datenbanken](#datenbanken)
- [Docker](#docker)
- [Virtuelle Maschinen](#virtuelle-maschinen)
- [Anwendungen](#anwendungen)
- [Best Practices](#best-practices)
<!-- /TOC -->

## Backup vs. Export

| | Backup | Export |
| --- | --- | --- |
| Ziel | Wiederherstellung **desselben** Systems | Übertragung / Migration / Austausch |
| Format | oft proprietär oder binär | offen: SQL, CSV, JSON, YAML, XML, OVF |
| Versionsabhängig | häufig ja (Restore nur in gleiche Version) | meist versionsunabhängig |
| Vollständig | ja (inkl. Rechte, Metadaten) | oft nur die Nutzdaten |
| Typischer Einsatz | Ausfall, Fehler, Ransomware | Umzug auf neuen Server, Upgrade, Weitergabe an andere, Analyse |

Ein guter Plan nutzt beides: **Backup** für den schnellen Restore, **Export** für die Unabhängigkeit von einem bestimmten System oder einer Version.

---

## Datenbanken

```bash
# MariaDB/MySQL – SQL-Export (ist zugleich der übliche Backup-Weg)
mysqldump --single-transaction smarthome > smarthome.sql
mysql smarthome < smarthome.sql

# Einzelne Tabelle als CSV (für Excel, Python, Auswertungen)
mysql -e "SELECT * FROM measurements" --batch smarthome | tr '\t' ',' > measurements.csv

# PostgreSQL – Tabelle als CSV
psql -d smarthome -c "\copy measurements TO 'measurements.csv' CSV HEADER"
psql -d smarthome -c "\copy measurements FROM 'measurements.csv' CSV HEADER"

# SQLite
sqlite3 data.db .dump > data.sql
sqlite3 new.db < data.sql
```

Für Messdaten aus Zeitreihendatenbanken (InfluxDB) ist ein **CSV- oder Line-Protocol-Export** oft sinnvoller als das native Backup, weil er sich mit jeder Software weiterverarbeiten lässt.

---

## Docker

Docker kennt zwei ähnlich klingende Befehlspaare, die oft verwechselt werden:

| Befehl | Was | Ergebnis |
| --- | --- | --- |
| `docker save` / `docker load` | **Image** inkl. aller Schichten, Tags und Historie | Image auf einem anderen Rechner ohne Registry verfügbar machen |
| `docker export` / `docker import` | **Dateisystem eines Containers**, flach, ohne Historie und ohne Volumes | Selten sinnvoll, verliert Metadaten (CMD, ENV, Ports) |

```bash
docker save -o beamer-ctl_1.2.tar beamer-ctl:1.2
docker load -i beamer-ctl_1.2.tar
```

**Volumes** sind in beiden Fällen **nicht** enthalten – die sichert man separat (siehe [Backups nach Domäne](Backups%20nach%20Domäne.md#docker)).

---

## Virtuelle Maschinen

| Plattform | Export | Import |
| --- | --- | --- |
| **Proxmox** | `vzdump <vmid>` → `.vma.zst` | `qmrestore <datei> <neue-vmid>` (VM), `pct restore` (LXC) |
| **VirtualBox / VMware** | OVF/OVA-Export (offenes Format) | OVF/OVA-Import |
| **Disk-Image** | `qemu-img convert -O qcow2 disk.raw disk.qcow2` | `qm importdisk <vmid> disk.qcow2 <storage>` |

**OVA** (*Open Virtual Appliance*) ist ein tar-Archiv aus OVF-Beschreibung (XML) und Festplattenabbildern. Damit lassen sich VMs zwischen Hypervisoren austauschen – z. B. eine vorbereitete Labor-VM für Studierende.

---

## Anwendungen

| Anwendung | Export | Import |
| --- | --- | --- |
| **openHAB** | `openhab-cli backup` (ZIP), in der UI: Code-Tab eines Things/Items/einer Regel (YAML) | `openhab-cli restore`, Code-Tab einfügen |
| **Grafana** | Dashboard → *Share → Export → JSON* | *Dashboards → Import* |
| **Node-RED** | *Menü → Export* (JSON) bzw. `~/.node-red/flows.json` | *Menü → Import* |
| **Home Assistant** | Backup (tar) | Restore |
| **Mosquitto** | Konfigurationsdateien (`/etc/mosquitto/`) | zurückkopieren |
| **Router / FRITZ!Box** | Konfigurationsdatei (passwortgeschützt) | Wiederherstellen |
| **Browser** | Lesezeichen als HTML | Import |
| **Passwort-Manager** | verschlüsselter Export (z. B. KeePass `.kdbx`) | Import |

---

## Best Practices

* **Exporte versionieren:** JSON/YAML-Exporte (Grafana, Node-RED, openHAB) in Git ablegen – dann sieht man im Diff, was sich geändert hat.
* **Encoding und Trennzeichen festlegen:** CSV immer in UTF-8, Trennzeichen dokumentieren (`,` vs. `;` – deutsches Excel erwartet oft `;`).
* **Import testen:** Ein Export ist erst dann etwas wert, wenn der Import auf einem frischen System funktioniert hat.
* **Geheimnisse beachten:** Exporte enthalten oft Passwörter, Tokens oder API-Keys im Klartext (z. B. Node-RED-Flows mit Credentials, openHAB-Things). Nicht ungeprüft weitergeben oder öffentlich committen.
* **Versionen notieren:** Mit welcher Programmversion wurde exportiert? Steht im Dateinamen oder in einer `README`.

---
