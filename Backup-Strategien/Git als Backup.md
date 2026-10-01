# 🌿 Git als Backup

Git ist ein **Versionskontrollsystem** – aber für Textdateien ist es zugleich ein hervorragendes Backup-Werkzeug: Jede Version ist gespeichert, jede Änderung nachvollziehbar, und ein `git push` auf einen entfernten Server ist eine Kopie außer Haus.

<!-- TOC -->
## Inhaltsverzeichnis

- [Wofür Git als Backup gut ist](#wofür-git-als-backup-gut-ist)
- [Konfiguration versionieren](#konfiguration-versionieren)
- [etckeeper: /etc automatisch in Git](#etckeeper-etc-automatisch-in-git)
- [Automatischer Commit per Cron](#automatischer-commit-per-cron)
- [Geheimnisse in Git](#geheimnisse-in-git)
- [Repositories selbst sichern](#repositories-selbst-sichern)
- [Grenzen](#grenzen)
<!-- /TOC -->

## Wofür Git als Backup gut ist

| Gut geeignet | Ungeeignet |
| --- | --- |
| Konfigurationsdateien (`/etc`, openHAB `conf/`, Mosquitto, Nginx) | Datenbanken (Binärdateien, ändern sich ständig) |
| Skripte, Ansible-Repository, Docker-Compose-Dateien | Große Binärdateien (Videos, Images, ISOs) |
| Dokumentation, LaTeX, Markdown | Logs |
| Dotfiles (`.bashrc`, `.vimrc`, `.ssh/config`) | Geheimnisse im Klartext (Passwörter, private Schlüssel!) |

Vorteile gegenüber einem `tar`-Backup:

* **Historie mit Begründung:** `git log` zeigt, wer wann was warum geändert hat.
* **Diff:** `git diff` zeigt genau, was sich seit gestern geändert hat – ideal, wenn „plötzlich nichts mehr geht“.
* **Gezielter Rollback:** Eine einzelne Datei auf einen alten Stand zurücksetzen: `git restore --source=HEAD~3 items/beamer.items`
* **Verteilt:** Jeder Klon ist eine vollständige Kopie inklusive Historie.

---

## Konfiguration versionieren

```bash
cd /etc/openhab
sudo git init
sudo git add .
sudo git commit -m "Initial import of openHAB configuration"
sudo git remote add origin git@github.com:<user>/openhab-config.git   # privates Repo!
sudo git push -u origin main
```

Eine `.gitignore` verhindert, dass Geheimnisse oder Laufzeitdaten im Repo landen:

```gitignore
# Secrets
*.key
*.pem
*.pass
secrets/
services/*.cfg.secret

# Runtime data
*.log
tmp/
```

---

## etckeeper: `/etc` automatisch in Git

[`etckeeper`](https://etckeeper.branchable.com/) legt `/etc` in ein Git-Repository und committet **automatisch** vor und nach jeder Paketinstallation sowie täglich per Cron.

```bash
sudo apt install etckeeper
sudo etckeeper init      # passiert bei Debian/Ubuntu meist automatisch
sudo etckeeper commit "Before changing mosquitto config"
cd /etc && sudo git log --oneline
```

> ⚠️ `/etc` enthält Geheimnisse (`/etc/shadow`, SSH-Host-Keys, Zertifikatsschlüssel). etckeeper schützt die Rechte lokal, aber ein `push` auf einen fremden Server sollte nur in ein **privates, vertrauenswürdiges** Repository oder verschlüsselt erfolgen.

---

## Automatischer Commit per Cron

Für Verzeichnisse, die auch über eine Weboberfläche geändert werden (z. B. openHAB-Regeln in der UI, die als Datei abgelegt werden), kann ein Skript Änderungen regelmäßig committen:

```bash
#!/usr/bin/env bash
# /usr/local/bin/git_autocommit.sh – commits and pushes changes in a directory
set -euo pipefail

REPO_DIR=${1:?Usage: git_autocommit.sh <repo-dir>}
cd "$REPO_DIR"

if [[ -n "$(git status --porcelain)" ]]; then
  git add -A
  git commit -q -m "Auto-commit on $(hostname) at $(date '+%F %T')"
  git push -q origin HEAD
  echo "Changes committed and pushed."
fi
```

```cron
# Jede Nacht um 01:00
0 1 * * * /usr/local/bin/git_autocommit.sh /etc/openhab >> /var/log/git_autocommit.log 2>&1
```

Auto-Commits ersetzen keine sinnvollen, manuellen Commits mit guter Nachricht. Sie sind das **Sicherheitsnetz** für vergessene Commits.

---

## Geheimnisse in Git

Geheimnisse gehören grundsätzlich **nicht** in Git. Wenn doch nötig:

| Werkzeug | Prinzip |
| --- | --- |
| **ansible-vault** | Verschlüsselt YAML-Dateien bzw. einzelne Werte für Ansible |
| **git-crypt** | Verschlüsselt ausgewählte Dateien transparent beim Commit |
| **SOPS** | Verschlüsselt Werte in YAML/JSON, Schlüssel z. B. über age oder GPG |

Wurde ein Passwort **versehentlich committet**: Das Passwort **sofort ändern**. Es aus der Historie zu entfernen (`git filter-repo`) reicht nicht – es könnte bereits geklont worden sein.

---

## Repositories selbst sichern

GitHub/GitLab sind kein Backup **deines** Codes, wenn der Account gesperrt, gelöscht oder das Repo versehentlich gelöscht wird. Zusätzlich:

```bash
# Vollständiger Spiegel inkl. aller Branches und Tags
git clone --mirror git@github.com:<user>/<repo>.git
cd <repo>.git && git remote update   # später aktualisieren

# Ein einziges Archiv mit kompletter Historie
git bundle create repo_$(date +%F).bundle --all
git clone repo_2026-10-01.bundle restored-repo   # Wiederherstellung
```

Beides lässt sich per Cron regelmäßig ausführen und auf das NAS legen (siehe [Tar & NAS](Tar%20&%20NAS.md)).

---

## Grenzen

* **Kein Backup bei fehlendem Push:** Ein lokales Repo stirbt mit der Festplatte.
* **Git vergisst nichts:** Große Dateien bleiben in der Historie und blähen das Repo dauerhaft auf. Für große Binärdateien gibt es **Git LFS**.
* **Keine Rechte und Besitzer:** Git speichert nur das Ausführbar-Bit, nicht Besitzer, Gruppe oder ACLs (etckeeper speichert diese zusätzlich in einer Metadatei).
* **Leere Verzeichnisse** werden nicht gespeichert (Konvention: `.gitkeep`).

---
