# 🚑 Wiederherstellung & Tests

Niemand braucht Backups. **Alle brauchen Wiederherstellungen.** Ein Backup ist erst dann etwas wert, wenn nachgewiesen ist, dass man daraus in angemessener Zeit einen funktionierenden Zustand herstellen kann.

<!-- TOC -->
## Inhaltsverzeichnis

- [Warum Backups scheitern](#warum-backups-scheitern)
- [Regelmäßige Restore-Tests](#regelmäßige-restore-tests)
- [Wiederherstellung in der Praxis](#wiederherstellung-in-der-praxis)
- [Monitoring](#monitoring)
- [Notfallhandbuch](#notfallhandbuch)
<!-- /TOC -->

## Warum Backups scheitern

Typische Gründe, warum im Ernstfall „das Backup nicht geht“:

* Der Backup-Job ist seit Monaten fehlgeschlagen, niemand hat es bemerkt.
* Das NAS war nicht gemountet, gesichert wurde auf die lokale Platte, die jetzt defekt ist.
* Die Datenbank wurde als Rohdatei im laufenden Betrieb kopiert und ist inkonsistent.
* Ein Inkrement in der Mitte der Kette fehlt oder ist beschädigt.
* Das Verschlüsselungspasswort des Backups lag nur auf dem ausgefallenen Server.
* Es wurde das Falsche gesichert (Daten liegen in einem Docker-Volume, gesichert wurde nur der Projektordner).
* Niemand weiß mehr, wie die Wiederherstellung funktioniert.

---

## Regelmäßige Restore-Tests

| Was | Wie oft | Wie |
| --- | --- | --- |
| Archive lesbar | bei jedem Backup (automatisch) | `tar -tf`, `sha256sum -c`, `borg check` |
| Einzelne Datei wiederherstellen | monatlich | Zufällige Datei aus dem Backup holen und mit dem Original vergleichen |
| Datenbank-Dump einspielen | monatlich / quartalsweise | In einen Test-Container importieren, Zeilen zählen |
| Komplettes System | halbjährlich | VM aus Backup als **neue** VM-ID wiederherstellen und starten (Netzwerk getrennt!) |
| Notfallplan durchspielen | jährlich | Eine **andere Person** stellt nur mit der Dokumentation wieder her |

Beispiel automatischer Datenbank-Test mit Docker:

```bash
#!/usr/bin/env bash
set -euo pipefail
DUMP=$(ls -1t /mnt/nas/backups/db/*.sql.zst | head -1)

docker run -d --name restore-test -e MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=1 mariadb:11
sleep 20
zstdcat "$DUMP" | docker exec -i restore-test mariadb
ROWS=$(docker exec restore-test mariadb -N -e "SELECT COUNT(*) FROM smarthome.measurements")
docker rm -f restore-test

echo "Restore test with $DUMP: $ROWS rows"
[[ "$ROWS" -gt 0 ]] || { echo "Restore test FAILED" >&2; exit 1; }
```

---

## Wiederherstellung in der Praxis

**Grundregeln:**

1. **Ruhe bewahren, nichts überschreiben.** Erst analysieren, was kaputt ist. Vom defekten System ggf. selbst noch eine Kopie ziehen.
2. **Nicht über das Original wiederherstellen**, sondern in ein separates Verzeichnis (`/tmp/restore`) oder eine neue VM. Dann vergleichen und gezielt zurückkopieren.
3. **Den richtigen Stand wählen:** Bei Ransomware oder schleichenden Fehlern ist das letzte Backup womöglich schon betroffen.
4. **Nach dem Restore prüfen:** Dienst startet, Daten sind vollständig, Clients verbinden sich.
5. **Dokumentieren:** Was ist passiert, was wurde wiederhergestellt, was lernen wir daraus?

**Beispiele:**

```bash
# Proxmox: VM als NEUE VM-ID wiederherstellen (Original bleibt unangetastet)
qmrestore /mnt/pve/nas-backup/dump/vzdump-qemu-101-2026_10_01-02_30_00.vma.zst 9101 --storage local-lvm

# Einzelne Datei aus tar holen
tar --zstd -xf weekly_2026-W40.tar.zst -C /tmp/restore etc/mosquitto/acl
diff /tmp/restore/etc/mosquitto/acl /etc/mosquitto/acl

# Datei aus Git wiederherstellen
git -C /etc/openhab log --oneline -- items/beamer.items
git -C /etc/openhab restore --source=<commit> items/beamer.items
```

---

## Monitoring

* **Erfolg melden statt Fehler melden:** Ein Healthcheck-Dienst (z. B. selbst gehostetes *Healthchecks*, Uptime Kuma) erwartet regelmäßig ein Signal. Bleibt es aus, gibt es Alarm – auch wenn das Skript gar nicht erst startet.
* **Alter des neuesten Backups prüfen:**

```bash
NEWEST=$(find /mnt/nas/backups -name '*.tar.zst' -mtime -8 | wc -l)
[[ "$NEWEST" -gt 0 ]] || echo "WARNING: no backup newer than 8 days!"
```

* **Füllstand** des Backup-Speichers überwachen (`df -h /mnt/nas`).
* **Größe der Backups** beobachten: Ein Backup, das plötzlich nur 2 KB statt 2 GB groß ist, ist verdächtig.

---

## Notfallhandbuch

Für jedes wichtige System gehört in die [Dokumentation](../Best%20Practices/Dokumentation.md) ein Abschnitt **„Restore“**:

* Wo liegen die Backups (Pfad, NAS, offsite)?
* Wie heißen die Dateien, welches ist das richtige?
* Wo liegen Passwörter/Schlüssel für verschlüsselte Backups (**nicht** nur auf dem gesicherten System)?
* Schritt-für-Schritt-Befehle zur Wiederherstellung.
* Wie prüft man, dass es funktioniert hat?
* Wer ist Ansprechpartner?

Dieses Handbuch muss auch dann erreichbar sein, wenn das System selbst ausgefallen ist (Ausdruck, zweites Repository, Wiki auf anderem Server).

---
