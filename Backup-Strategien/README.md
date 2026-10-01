# Backup-Strategien

> **Ein Backup, das nie wiederhergestellt wurde, ist kein Backup – sondern eine Hoffnung.**

Dieser Ordner erklärt, welche Arten von Backups es gibt, wie man unterschiedliche Systeme (VMs, Container, Datenbanken, Konfigurationen) sichert, wie man Backups automatisiert, wie lange man sie aufhebt und wie man sie wiederherstellt.

<!-- TOC -->
## Inhaltsverzeichnis

- [Übersicht](#übersicht)
- [Die fünf wichtigsten Regeln](#die-fünf-wichtigsten-regeln)
<!-- /TOC -->

## Übersicht

| Dokument | Inhalt |
| --- | --- |
| [Backup-Arten](Backup-Arten.md) | Voll, inkrementell, differentiell, Snapshot, Image, Sync – und warum RAID kein Backup ist. 3-2-1-Regel, RPO/RTO |
| [Backups nach Domäne](Backups%20nach%20Domäne.md) | Was „Backup“ bei VMs, Containern, Datenbanken, Dateien, Netzwerkgeräten und openHAB bedeutet |
| [Git als Backup](Git%20als%20Backup.md) | Konfigurationen versionieren, `etckeeper`, Auto-Commit, Grenzen |
| [Backup-Skripte & Intervalle](Backup-Skripte%20&%20Intervalle.md) | Wöchentliche, monatliche und jährliche Skripte mit Cron |
| [Tar & NAS](Tar%20&%20NAS.md) | Archive mit `tar`, Übertragung mit `rsync`, NAS per NFS/SMB einbinden |
| [Retention & Rotation](Retention%20&%20Rotation.md) | Alte Backups löschen: „die letzten N“, Großvater-Vater-Sohn, Monats-Voll + Wochen-Inkrementell |
| [Export & Import](Export%20&%20Import.md) | Datenbank-Dumps, Docker/Proxmox-Exporte, openHAB, Grafana, Node-RED |
| [Backups mit Ansible](Backups%20mit%20Ansible.md) | Backups zentral ausrollen und einsammeln; Infrastructure as Code als Backup |
| [Wiederherstellung & Tests](Wiederherstellung%20&%20Tests.md) | Restore üben, Prüfsummen, Monitoring, Notfallhandbuch |

## Die fünf wichtigsten Regeln

1. **3-2-1:** Drei Kopien, auf zwei verschiedenen Medien, eine davon außer Haus.
2. **Automatisieren:** Ein Backup, das man von Hand starten muss, wird vergessen.
3. **Konsistent sichern:** Laufende Datenbanken und VMs nicht einfach kopieren.
4. **Aufräumen (Retention):** Sonst ist irgendwann die Platte voll – und das Backup schlägt still fehl.
5. **Wiederherstellung testen:** Regelmäßig, nach Plan, mit Protokoll.
