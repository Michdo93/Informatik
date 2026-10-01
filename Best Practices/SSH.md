# 🔑 Best Practice: SSH

**Regeln in Kurzform:**

1. SSH läuft **nicht auf dem Default-Port 22**, sondern auf einem eigenen, dokumentierten Port.
2. Anmeldung grundsätzlich mit **Public-/Private-Key**.
3. Zusammen mit SSH wird **`sshpass`** installiert – für die Fälle, in denen Programme (Ansible, openHAB Exec-Binding, Skripte) nicht-interaktiv mit Passwort zugreifen müssen.
4. Direkter Root-Login ist verboten.

<!-- TOC -->
## Inhaltsverzeichnis

- [Installation](#installation)
- [Eigener Port statt Port 22](#eigener-port-statt-port-22)
- [Public-/Private-Key](#public-private-key)
- [SSH-Client-Konfiguration](#ssh-client-konfiguration)
- [sshpass](#sshpass)
- [Dateien übertragen: scp, rsync, sftp](#dateien-übertragen-scp-rsync-sftp)
- [Weitere Härtung](#weitere-härtung)
- [Checkliste](#checkliste)
<!-- /TOC -->

## Installation

```bash
sudo apt update
sudo apt install openssh-server sshpass
sudo systemctl enable --now ssh
```

---

## Eigener Port statt Port 22

Auf Port 22 laufen im Internet ununterbrochen automatisierte Login-Versuche. Ein anderer Port hält den Großteil dieser Bots fern, die Logs bleiben lesbar und es fällt auf, wenn wirklich jemand gezielt sucht.

> Wichtig: Ein anderer Port ist **keine** Sicherheitsmaßnahme im eigentlichen Sinn („Security by Obscurity“). Ein Portscan findet ihn trotzdem. Er **ergänzt** Schlüssel-Login und Firewall, er ersetzt sie nicht.

`/etc/ssh/sshd_config.d/10-custom.conf`:

```text
Port 2222
```

Bei neueren Ubuntu-Versionen (ab 22.10) wird SSH über **Socket Activation** gestartet. Dann muss der Port zusätzlich im Socket geändert werden:

```bash
sudo systemctl edit ssh.socket
# [Socket]
# ListenStream=
# ListenStream=2222
sudo systemctl daemon-reload
sudo systemctl restart ssh.socket
```

Firewall öffnen **bevor** die alte Verbindung getrennt wird:

```bash
sudo ufw allow 2222/tcp
sudo sshd -t && sudo systemctl restart ssh
# In einem ZWEITEN Terminal testen, erst dann das erste schließen:
ssh -p 2222 user@host
```

Der Port wird pro Netz/Labor einheitlich gewählt und dokumentiert (z. B. im Ansible-Inventar als `ansible_port`).

---

## Public-/Private-Key

**Schlüssel erzeugen** (einmal pro Person und Rechner, Ed25519 ist aktueller Standard):

```bash
ssh-keygen -t ed25519 -C "max.mustermann@hs-furtwangen.de"
```

* Privater Schlüssel: `~/.ssh/id_ed25519` – bleibt auf dem eigenen Rechner, mit Passphrase schützen
* Öffentlicher Schlüssel: `~/.ssh/id_ed25519.pub` – wird auf die Server verteilt

**Öffentlichen Schlüssel auf den Server kopieren:**

```bash
ssh-copy-id -p 2222 -i ~/.ssh/id_ed25519.pub user@host
```

**Danach Passwort-Login für Menschen abschalten** (`/etc/ssh/sshd_config.d/10-custom.conf`):

```text
Port 2222
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
MaxAuthTries 3
AllowUsers admin ansible
```

Für Dienst-Accounts, die per `sshpass` mit Passwort zugreifen müssen, kann der Passwort-Login gezielt pro Benutzer oder Quellnetz erlaubt werden:

```text
Match User openhab-exec Address 192.168.1.0/24
    PasswordAuthentication yes
```

---

## SSH-Client-Konfiguration

Damit man nicht jedes Mal Port, User und Schlüssel tippen muss: `~/.ssh/config`

```text
Host pi-beamer
    HostName 192.168.1.51
    Port 2222
    User admin
    IdentityFile ~/.ssh/id_ed25519

Host lab-*
    Port 2222
    User admin
```

Danach genügt `ssh pi-beamer` bzw. `scp datei pi-beamer:/tmp/`.

---

## sshpass

`sshpass` übergibt ein Passwort **nicht-interaktiv** an SSH. Das wird gebraucht, wenn ein Programm einen Befehl auf einem anderen Rechner ausführen soll, aber niemand da ist, der das Passwort eintippt.

**Typische Einsatzfälle:**

| Fall | Warum `sshpass` |
| --- | --- |
| **Ansible** mit Passwort-Login (`ansible_password`, `--ask-pass`, Erst-Einrichtung eines Geräts, bevor Schlüssel verteilt sind) | Ansible nutzt intern `sshpass`, wenn per Passwort verbunden wird. Fehlt das Paket, bricht Ansible mit einer Fehlermeldung ab. |
| **openHAB Exec-Binding** | openHAB führt als Benutzer `openhab` einen Befehl aus, z. B. Herunterfahren eines anderen Rechners. Es gibt kein Terminal für eine Passworteingabe. |
| **Cron-Jobs, Skripte, Geräte ohne Key-Unterstützung** | Manche Embedded-Geräte und Switches erlauben nur Passwort-Login. |

**Sicher verwenden** – das Passwort **nicht** mit `-p` auf der Kommandozeile übergeben, denn dann steht es in der Prozessliste (`ps aux`) und in der Shell-History:

```bash
# Schlecht – Passwort sichtbar in ps und History:
sshpass -p 'geheim' ssh -p 2222 user@host 'uptime'

# Besser – Passwort aus Datei (Rechte 600):
sshpass -f ~/.ssh/host.pass ssh -p 2222 user@host 'uptime'

# Oder aus der Umgebungsvariable SSHPASS:
export SSHPASS='geheim'
sshpass -e ssh -p 2222 user@host 'uptime'
```

**Beispiel openHAB Exec-Binding:** Befehl in `exec.whitelist` eintragen (siehe [Zugriffskontrolle → Whitelist & Blacklist](../Zugriffskontrolle/Whitelist%20&%20Blacklist.md)):

```text
sshpass -f /etc/openhab/secrets/nas.pass ssh -p 2222 -o StrictHostKeyChecking=accept-new admin@192.168.1.20 sudo /sbin/poweroff
```

> **Abwägung:** Wo immer es geht, ist ein **eigener SSH-Schlüssel ohne Passphrase für den Dienst-Account** (z. B. `openhab`) mit eingeschränkten Rechten die sicherere Variante, weil dann gar kein Passwort-Login auf dem Zielsystem nötig ist. `sshpass` ist das Werkzeug für alle Fälle, in denen das nicht möglich oder (noch) nicht eingerichtet ist. Deshalb gehört es bei jeder SSH-Installation mit dazu.

---

## Dateien übertragen: scp, rsync, sftp

Über SSH lassen sich auch Dateien sicher kopieren. Wichtig: Bei **`scp`** wird der Port mit **großem `-P`** angegeben, bei `ssh` mit kleinem `-p`.

```bash
# local -> remote
scp -P 2222 config.yaml pi@192.168.10.21:/opt/lab/app/
# remote -> local
scp -P 2222 pi@192.168.10.21:/var/log/syslog ./syslog-pi.txt
# whole directory
scp -P 2222 -r ./dist pi@192.168.10.21:/opt/lab/app/
# with an alias from ~/.ssh/config (port, user, key are taken from there)
scp config.yaml pi-mqtt:/opt/lab/app/
```

**rsync** überträgt nur **Änderungen**, kann abgebrochene Übertragungen fortsetzen und Rechte erhalten – ideal für größere Verzeichnisse, Deployments und Backups (→ [Tar & NAS](../Backup-Strategien/Tar%20%26%20NAS.md)):

```bash
rsync -avz --delete -e "ssh -p 2222" ./app/ pi@192.168.10.21:/opt/lab/app/
#      │││   │
#      │││   └─ delete files on the target that no longer exist locally (careful!)
#      ││└─ compress during transfer
#      │└─ verbose
#      └─ archive: recursive, keep permissions, times, symlinks
rsync -avzn ...     # -n = dry run: show what would happen
```

> Auf den **abschließenden Schrägstrich** achten: `./app/` kopiert den **Inhalt** von `app`, `./app` den **Ordner selbst** in das Ziel.

**sftp** bietet eine interaktive Sitzung (`sftp -P 2222 pi@host`, dann `put`, `get`, `ls`); grafische Clients wie **FileZilla** oder **WinSCP** nutzen dasselbe Protokoll.

Mit `sshpass` funktionieren auch `scp` und `rsync` nicht-interaktiv: `sshpass -f ~/.ssh/host.pass scp -P 2222 datei user@host:/ziel/`.

---

## Weitere Härtung

* **Fail2ban** sperrt IPs nach mehreren fehlgeschlagenen Logins: `sudo apt install fail2ban`
* **Firewall**: SSH nur aus dem Labor-/Verwaltungsnetz erlauben (`ufw allow from 192.168.1.0/24 to any port 2222 proto tcp`)
* **Host-Keys prüfen**: Bei der Meldung `REMOTE HOST IDENTIFICATION HAS CHANGED!` nicht blind `ssh-keygen -R` ausführen, sondern klären, ob das Gerät neu aufgesetzt wurde.
* **`sudo` statt Root**: Eigener Admin-Benutzer mit `sudo`-Rechten, für Dienst-Accounts nur die nötigen Befehle in `/etc/sudoers.d/` freigeben.
* **Keine Schlüssel teilen**: Jede Person und jeder Dienst hat einen eigenen Schlüssel – so kann man einzelne Zugänge wieder entziehen.

---

## Checkliste

- [ ] `openssh-server` und `sshpass` installiert
- [ ] Eigener Port konfiguriert, Firewall angepasst, Port dokumentiert
- [ ] Public Key hinterlegt, Login mit Schlüssel getestet
- [ ] `PermitRootLogin no`, Passwort-Login nur wo nötig (per `Match`)
- [ ] Fail2ban aktiv
- [ ] Gerät in `~/.ssh/config` und im [Ansible](Ansible.md)-Inventar eingetragen

---
