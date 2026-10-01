# 🐧 Linux & Werkzeuge

Praktisches Wissen rund um Linux-Systeme, wie sie im Labor überall vorkommen: Raspberry Pis, Server, VMs und Container.

<!-- TOC -->
## Inhaltsverzeichnis

- [Übersicht](#übersicht)
- [Verwandte Dokumente](#verwandte-dokumente)
<!-- /TOC -->

## Übersicht

| Dokument | Inhalt |
| --- | --- |
| [systemd-Services](systemd-Services.md) | Eigene Service-Units: Typen (`simple`, `forking`, `oneshot` …), `ExecStartPre`/`ExecStart`/`ExecStop`, Abhängigkeiten, verzögerter Start, Benutzer, Restart-Limits, Logging |
| [Bash-Skripte](Bash-Skripte.md) | Robuste Skripte: Shebang, Strict Mode, Quoting, Bedingungen, Schleifen, Funktionen, Exit-Codes, `trap`, ShellCheck |
| [Benutzer, Gruppen & Rechte](Benutzer%2C%20Gruppen%20%26%20Rechte.md) | Benutzer und Gruppen verwalten, `rwx` und Oktalschreibweise, `chmod`/`chown`, SGID, umask, `sudo`, POSIX-ACLs |
| [Cron & systemd-Timer](Cron%20%26%20systemd-Timer.md) | Zeitgesteuerte Aufgaben: Crontab-Syntax, Cron-Job vs. Cron-Task, Fallen, anacron, systemd-Timer, Verwaltung mit Ansible |

---

## Verwandte Dokumente

* [Best Practice SSH](../Best%20Practices/SSH.md) – Zugriff auf entfernte Systeme
* [Best Practice Ansible](../Best%20Practices/Ansible.md) – viele Systeme gleichzeitig verwalten
* [Backup-Skripte & Intervalle](../Backup-Strategien/Backup-Skripte%20%26%20Intervalle.md) – Skripte für Cron
* [Bootstrapping](../Software-Konzepte/Bootstrapping.md) – was beim Booten passiert
* [Cross-Plattform-Tricks](../Workarounds%20%26%20Hacks/Cross-Plattform-Tricks.md) – Linux vs. Windows vs. macOS

---
