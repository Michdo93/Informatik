# 👥 Benutzer, Gruppen & Rechte

Linux ist von Grund auf ein **Mehrbenutzersystem**. Wer darf eine Datei lesen, wer sie ändern, wer ein Programm ausführen? Das regeln **Benutzer**, **Gruppen** und **Dateirechte**. Im Labor ist das wichtig, damit Dienste nicht als `root` laufen, Passwörter nicht für jeden lesbar sind und mehrere Personen sauber an denselben Systemen arbeiten können.

Die konzeptionelle Einordnung (Authentifizierung, Autorisierung, ACLs in anderen Domänen) steht unter [Zugriffskontrolle](../Zugriffskontrolle/README.md).

<!-- TOC -->
## Inhaltsverzeichnis

- [Benutzer](#benutzer)
  - [Benutzer anlegen und verwalten](#benutzer-anlegen-und-verwalten)
  - [Wichtige Dateien](#wichtige-dateien)
- [Gruppen](#gruppen)
- [Dateirechte](#dateirechte)
  - [Rechte lesen](#rechte-lesen)
  - [Was bedeuten r, w, x?](#was-bedeuten-r-w-x)
  - [Oktalschreibweise](#oktalschreibweise)
- [Rechte ändern](#rechte-ändern)
  - [chmod](#chmod)
  - [chown und chgrp](#chown-und-chgrp)
- [Spezialrechte](#spezialrechte)
- [umask](#umask)
- [sudo](#sudo)
- [Erweiterte ACLs](#erweiterte-acls)
- [Typische Laborszenarien](#typische-laborszenarien)
- [Checkliste](#checkliste)
<!-- /TOC -->

## Benutzer

Jeder Benutzer hat einen Namen und eine eindeutige **UID** (User ID).

| Art | UID (Debian/Ubuntu) | Beispiel |
| --- | --- | --- |
| **root** | 0 | Administrator, darf alles |
| **Systembenutzer** | 1–999 | `www-data`, `mosquitto`, `openhab` – für Dienste, ohne Login |
| **Normale Benutzer** | ab 1000 | `pi`, `michael`, `hiwi` |

```bash
whoami                         # who am I?
id                             # UID, GID and groups of the current user
id openhab                     # ... of another user
getent passwd openhab          # entry from /etc/passwd
```

### Benutzer anlegen und verwalten

```bash
sudo adduser hiwi                       # interactive (Debian): home, password, ...
sudo useradd -m -s /bin/bash hiwi       # low level: -m creates home directory
sudo passwd hiwi                        # set password

# system user for a service: no home, no login shell
sudo useradd --system --no-create-home --shell /usr/sbin/nologin labservice

sudo usermod -aG sudo hiwi              # add to group 'sudo' (admin rights)
sudo usermod -L hiwi                    # lock account
sudo deluser --remove-home hiwi         # delete user incl. home directory
```

> ⚠️ Bei `usermod -G` **immer** `-a` (append) angeben. `usermod -G sudo hiwi` ohne `-a` entfernt den Benutzer aus **allen anderen** Gruppen.

Gruppenänderungen wirken erst nach **erneutem Anmelden** (oder `newgrp gruppe` in der aktuellen Shell).

### Wichtige Dateien

| Datei | Inhalt |
| --- | --- |
| `/etc/passwd` | Benutzer, UID, Home-Verzeichnis, Shell (für alle lesbar) |
| `/etc/shadow` | Passwort-Hashes (nur für root lesbar) |
| `/etc/group` | Gruppen und Mitglieder |
| `/etc/sudoers`, `/etc/sudoers.d/` | Wer darf was mit `sudo` |

---

## Gruppen

Gruppen bündeln Rechte für mehrere Benutzer. Auf einem Raspberry Pi steuern Gruppen z. B. den Zugriff auf Hardware:

| Gruppe | Zugriff auf |
| --- | --- |
| `sudo` | Administratorrechte via `sudo` |
| `dialout` | Serielle Schnittstellen (`/dev/ttyUSB0`, `/dev/ttyACM0`) |
| `gpio`, `i2c`, `spi` | GPIO-Pins und Busse am Raspberry Pi |
| `video` | Kamera, Grafik |
| `audio` | Soundgeräte |
| `docker` | Docker steuern – **entspricht faktisch root-Rechten!** |
| `adm` | Logdateien in `/var/log` lesen |

```bash
groups                                  # my groups
sudo groupadd labteam                   # create a group
sudo usermod -aG labteam,dialout hiwi   # add user to groups
sudo gpasswd -d hiwi labteam            # remove user from a group
```

> **Typischer Fehler:** `Permission denied: '/dev/ttyUSB0'` – der Benutzer ist nicht in der Gruppe `dialout`. Bei systemd-Diensten: `SupplementaryGroups=dialout` (→ [systemd-Services](systemd-Services.md)).

---

## Dateirechte

### Rechte lesen

```bash
ls -l
# -rwxr-x---  1 labservice labteam  2048 Oct  1 10:00 backup.sh
# drwxr-xr-x  2 root       root     4096 Oct  1 09:00 config
```

```text
 -   rwx   r-x   ---    labservice  labteam
 │    │     │     │         │          │
 │    │     │     │         │          └─ Gruppe
 │    │     │     │         └──────────── Eigentümer (owner)
 │    │     │     └────────────────────── Rechte für alle anderen (others)
 │    │     └──────────────────────────── Rechte für die Gruppe (group)
 │    └────────────────────────────────── Rechte für den Eigentümer (user)
 └─────────────────────────────────────── Typ: - Datei, d Verzeichnis, l Link
```

### Was bedeuten r, w, x?

| Recht | Bei Dateien | Bei Verzeichnissen |
| --- | --- | --- |
| **r** (read) | Inhalt lesen | Dateinamen auflisten (`ls`) |
| **w** (write) | Inhalt ändern | Dateien anlegen, **löschen**, umbenennen |
| **x** (execute) | Als Programm ausführen | Hineinwechseln (`cd`) und auf Dateien darin zugreifen |

> Ob man eine Datei **löschen** darf, hängt am **Schreibrecht des Verzeichnisses**, nicht an den Rechten der Datei selbst.

### Oktalschreibweise

Jede Rechtegruppe wird als Zahl aus **r = 4, w = 2, x = 1** zusammengezählt:

| Zahl | Rechte | Zahl | Rechte |
| --- | --- | --- | --- |
| 7 | `rwx` | 3 | `-wx` |
| 6 | `rw-` | 2 | `-w-` |
| 5 | `r-x` | 1 | `--x` |
| 4 | `r--` | 0 | `---` |

**Typische Werte:**

| Oktal | Symbolisch | Einsatz |
| --- | --- | --- |
| `755` | `rwxr-xr-x` | Programme, Skripte, Verzeichnisse |
| `750` | `rwxr-x---` | Skripte/Verzeichnisse nur für Eigentümer und Gruppe |
| `644` | `rw-r--r--` | Normale Dateien, Konfiguration ohne Geheimnisse |
| `640` | `rw-r-----` | Konfiguration, die nur der Dienst (Gruppe) lesen soll |
| `600` | `rw-------` | **Passwörter, private Schlüssel**, `.env`-Dateien |
| `700` | `rwx------` | Private Verzeichnisse, z. B. `~/.ssh` |

---

## Rechte ändern

### chmod

```bash
chmod 755 backup.sh             # octal
chmod +x backup.sh              # add execute for everyone (respecting umask)
chmod u+x,g-w,o= backup.sh      # symbolic: user +x, group -w, others nothing
chmod -R g+rw /opt/lab/shared   # recursive
```

| Symbol | Bedeutung |
| --- | --- |
| `u`, `g`, `o`, `a` | user, group, others, all |
| `+`, `-`, `=` | hinzufügen, entfernen, genau setzen |

**Rekursiv, aber unterschiedlich für Dateien und Verzeichnisse:**

```bash
find /opt/lab/app -type d -exec chmod 755 {} +
find /opt/lab/app -type f -exec chmod 644 {} +
chmod -R u=rwX,go=rX /opt/lab/app     # X = execute only for directories (and already executable files)
```

### chown und chgrp

```bash
sudo chown labservice backup.sh                 # change owner
sudo chown labservice:labteam backup.sh         # owner and group
sudo chown -R openhab:openhab /etc/openhab      # recursive
sudo chgrp labteam shared.txt                   # only group
```

---

## Spezialrechte

| Recht | Oktal | Bei Dateien | Bei Verzeichnissen |
| --- | --- | --- | --- |
| **SUID** | `4000` | Programm läuft mit den Rechten des **Eigentümers** (z. B. `passwd`) | – |
| **SGID** | `2000` | Programm läuft mit den Rechten der **Gruppe** | Neue Dateien erben die **Gruppe des Verzeichnisses** – ideal für Team-Ordner |
| **Sticky Bit** | `1000` | – | Jeder darf nur **eigene** Dateien löschen (z. B. `/tmp`) |

**Gemeinsames Projektverzeichnis für das Labor-Team:**

```bash
sudo mkdir -p /opt/lab/shared
sudo chown root:labteam /opt/lab/shared
sudo chmod 2775 /opt/lab/shared        # SGID: new files belong to 'labteam'
```

> **SUID** auf eigene Skripte oder Programme setzen ist ein **Sicherheitsrisiko** – dafür gibt es `sudo` mit gezielten Regeln.

---

## umask

Die **umask** legt fest, welche Rechte bei **neu angelegten** Dateien **entzogen** werden.

```bash
umask          # e.g. 0022 -> new files 644, new directories 755
umask 027      # new files 640, directories 750 (others get nothing)
```

Für systemd-Dienste: `UMask=0027` im Abschnitt `[Service]`.

---

## sudo

`sudo` führt einzelne Befehle mit Root-Rechten aus und **protokolliert** sie – besser als dauerhaft als `root` zu arbeiten.

```bash
sudo systemctl restart mosquitto
sudo -u openhab ls /var/lib/openhab     # run as another user
sudo -i                                 # root shell (use sparingly)
```

Regeln **immer** mit `visudo` bearbeiten – es prüft die Syntax. Ein Fehler in `sudoers` kann sonst jeden `sudo`-Zugang sperren.

```bash
sudo visudo -f /etc/sudoers.d/openhab
```

```text
# allow openHAB (e.g. Exec binding) to restart one specific service - nothing else
openhab ALL=(root) NOPASSWD: /usr/bin/systemctl restart beamer-control.service
```

So erhält ein Dienst genau die Rechte, die er braucht (**Least Privilege**), statt pauschal Root-Rechte.

---

## Erweiterte ACLs

Reichen Eigentümer/Gruppe/Andere nicht aus, kann man mit **POSIX-ACLs** einzelnen Benutzern oder Gruppen zusätzliche Rechte geben:

```bash
sudo apt install acl
setfacl -m u:hiwi:rw /opt/lab/config.yaml       # give user 'hiwi' rw
setfacl -m d:g:labteam:rwX /opt/lab/shared      # default ACL for new files
getfacl /opt/lab/config.yaml
```

Erkennbar am `+` in `ls -l`: `-rw-rw-r--+`. Mehr dazu unter [ACL](../Zugriffskontrolle/ACL.md).

---

## Typische Laborszenarien

| Szenario | Lösung |
| --- | --- |
| Python-Dienst soll auf `/dev/ttyUSB0` zugreifen | Dienstbenutzer in Gruppe `dialout`, kein `root` |
| `.env` mit MQTT-Passwort | `chmod 600`, Eigentümer = Dienstbenutzer |
| SSH-Schlüssel wird abgelehnt | `chmod 700 ~/.ssh`, `chmod 600 ~/.ssh/authorized_keys` und `~/.ssh/id_*`, Home-Verzeichnis nicht für andere schreibbar |
| openHAB kann eine Datei nicht lesen | `sudo -u openhab cat <datei>` testen, dann Gruppe oder ACL anpassen |
| Mehrere Hiwis arbeiten im selben Projektordner | Gruppe `labteam`, Verzeichnis mit SGID `2775` |
| Skript lässt sich nicht starten (`Permission denied`) | `chmod +x skript.sh` – oder Dateisystem mit `noexec` gemountet |
| Webserver liefert 403 | Nginx-Benutzer (`www-data`) braucht `x` auf **allen** Verzeichnissen des Pfads und `r` auf der Datei |

---

## Checkliste

* [ ] Jeder Dienst hat einen **eigenen Systembenutzer**, keiner läuft unnötig als `root`
* [ ] Geheimnisse (`.env`, Schlüssel, Zertifikats-Keys) haben Rechte `600`
* [ ] Persönliche Konten für Personen – kein gemeinsames Konto, dessen Passwort jeder kennt
* [ ] `sudo`-Regeln so eng wie möglich, nur über `visudo`
* [ ] Gemeinsame Ordner über Gruppen (+ SGID) statt `chmod 777`
* [ ] `chmod 777` kommt **nie** vor

---
