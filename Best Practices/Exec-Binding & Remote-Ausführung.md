# 🧰 Exec Binding & Remote-Ausführung

Mit dem **Exec Binding** (und der `executeCommandLine`-Aktion) kann openHAB **Kommandozeilenbefehle** ausführen. Das ist mächtig und verlockend – und zugleich eine der größten Sicherheitsfallen im Smart Home. Dieses Kapitel zeigt, wie man es **sicher** einsetzt und wann es bessere Alternativen gibt.

<!-- TOC -->
## Inhaltsverzeichnis

- [Was geht damit?](#was-geht-damit)
- [Die Whitelist ist Pflicht](#die-whitelist-ist-pflicht)
- [Die häufigsten Sicherheitsfehler](#die-häufigsten-sicherheitsfehler)
- [Remote-Rechner steuern](#remote-rechner-steuern)
  - [Linux per SSH](#linux-per-ssh)
  - [Windows per OpenSSH](#windows-per-openssh)
  - [Wake-on-LAN](#wake-on-lan)
- [Systeminfos abfragen – besser mit Bordmitteln](#systeminfos-abfragen--besser-mit-bordmitteln)
- [Eigene Skripte als Treiber](#eigene-skripte-als-treiber)
- [Checkliste](#checkliste)
<!-- /TOC -->

## Was geht damit?

Das Exec Binding führt Befehle mit den Rechten des openHAB-Benutzers aus. Typische Einsätze im Labor:

| Zweck | Beispiel |
| --- | --- |
| Systeminfos | `uptime`, `systemctl status`, Plattenplatz |
| Netzwerk | `ping`, `nmap`, Portcheck, Zertifikatsablauf |
| Remote-Rechner | per SSH ein-/ausschalten, Anwendungen starten |
| Wake-on-LAN | `etherwake`/`wakeonlan` |
| Eigene Skripte | Python-/Shell-Programme als Gerätetreiber |

> **Grundhaltung:** Das Exec Binding ist die **letzte Wahl**, nicht die erste. Für Gerätesteuerung ist meist **MQTT** oder ein kleiner Dienst sauberer, testbarer und sicherer. Exec ist sinnvoll für lokale Systemaktionen, für die es kein Binding gibt.

---

## Die Whitelist ist Pflicht

Aus Sicherheitsgründen führt openHAB nur Befehle aus, die in der Datei

```text
$OPENHAB_CONF/misc/exec.whitelist      # Debian/openHABian: /etc/openhab/misc/exec.whitelist
```

**exakt** (Zeichen für Zeichen) eingetragen sind. Jeder Befehl steht in einer eigenen Zeile.

```text
# every command must match exactly, including arguments
/etc/openhab/scripts/backup.sh
/etc/openhab/scripts/ping.sh %2$s
/usr/bin/wakeonlan b8:27:eb:12:34:56
```

`%2$s` ist ein Platzhalter für das Argument, das die Thing-Konfiguration übergibt. Mehr dazu und zur Einordnung von Whitelists steht unter [Whitelist & Blacklist](../Zugriffskontrolle/Whitelist%20%26%20Blacklist.md).

---

## Die häufigsten Sicherheitsfehler

Die folgenden Muster tauchen in vielen Altkonfigurationen auf – und sind **gefährlich**:

| Anti-Muster | Problem | Besser |
| --- | --- | --- |
| `sudo %2$s` in der Whitelist | Erlaubt **beliebige** Root-Befehle über openHAB | Nur **konkrete** Befehle: `sudo /bin/systemctl restart mosquitto.service` |
| `sshpass -p geheim …` | Passwort steht **im Klartext** in Konfiguration, Logs und Prozessliste | **SSH-Key** ohne Passwort für den Dienst; falls nötig `sshpass -f datei` (Datei mit Rechten `600`) |
| PsExec `-p passwort` (Windows) | Passwort im Klartext | Windows-Anmeldung per Schlüssel/gespeichertem Credential, kein Klartext |
| `StrictHostKeyChecking=no` dauerhaft | Macht Man-in-the-Middle möglich | Host-Key einmalig akzeptieren und in `known_hosts` festschreiben |
| openHAB-Benutzer in der `sudoers` ohne Einschränkung | Vollzugriff als root | `NOPASSWD` nur für einzelne, genau benannte Befehle |

**sudo gezielt freigeben** (mit `visudo`, siehe [Benutzer, Gruppen & Rechte](../Linux%20%26%20Werkzeuge/Benutzer%2C%20Gruppen%20%26%20Rechte.md)):

```text
# /etc/sudoers.d/openhab  — nur diese Befehle, nichts anderes
openhab ALL=(root) NOPASSWD: /bin/systemctl restart mosquitto.service
openhab ALL=(root) NOPASSWD: /usr/sbin/etherwake
```

---

## Remote-Rechner steuern

### Linux per SSH

```bash
# key-based, no password anywhere
ssh -o BatchMode=yes labservice@192.168.10.30 'sudo /sbin/poweroff'
```

Einrichtung: Schlüsselpaar für den openHAB-Dienst erzeugen (`ssh-keygen`), den öffentlichen Schlüssel auf den Zielrechner kopieren (`ssh-copy-id`), auf dem Ziel ein `sudoers`-Eintrag nur für `poweroff`/`reboot`. Für den Start grafischer Programme braucht es oft `DISPLAY=:0` und einen angemeldeten Benutzer.

### Windows per OpenSSH

Moderne Windows-Versionen bringen einen **OpenSSH-Server** mit (Features → „OpenSSH Server“). Damit entfällt das fehleranfällige PsExec in vielen Fällen:

```bash
ssh admin@192.168.10.40 "shutdown /s /t 0"
ssh admin@192.168.10.40 "start steam://rungameid/620980"
```

Wake-on-LAN zum Einschalten (im BIOS und im Netzwerktreiber aktivieren), geregeltes Herunterfahren per SSH. PsExec/`xdotool`/`runas` nur, wenn es keinen anderen Weg gibt – und nie mit Klartext-Passwort.

### Wake-on-LAN

```bash
# enable on the target (Linux)
sudo ethtool -s eth0 wol g
# send the magic packet from the openHAB host
wakeonlan b8:27:eb:12:34:56      # or: etherwake -i eth0 b8:27:eb:12:34:56
```

Voraussetzung: Das Gerät hat eine **feste IP/MAC** (→ [DHCP](DHCP.md)) und WoL ist in Firmware/BIOS und Treiber aktiviert.

---

## Systeminfos abfragen – besser mit Bordmitteln

In Altkonfigurationen werden Dienstinfos oft mühsam aus `systemctl status … | grep … | awk …` herausgeschnitten. Das ist fragil (bricht bei jeder Formatänderung). Robuster:

```bash
systemctl show mosquitto -p ActiveState --value     # active / inactive
systemctl show mosquitto -p SubState --value        # running / dead
systemctl show mosquitto -p MainPID --value
systemctl is-active mosquitto                        # exit code + text
```

Noch besser ist, solche Werte **gar nicht** über Exec zu holen, sondern über das **systeminfo-Binding**, das **network-Binding** oder ein richtiges [Monitoring](Monitoring%20%26%20Alerting.md) (Nagios/NRPE, Prometheus-Exporter).

---

## Eigene Skripte als Treiber

Wenn ein Gerät kein Binding hat, ist ein kleines **eigenes Programm** oft besser als eine Kette von Exec-Befehlen:

* Das Programm spricht das Gerät an (seriell, TCP, HTTP, Bluetooth) und bindet sich per **MQTT** an openHAB.
* Es läuft als **systemd-Service** (→ [systemd-Services](../Linux%20%26%20Werkzeuge/systemd-Services.md)), nicht als Exec-Aufruf.
* Vorteil: testbar, protokolliert, überlebt openHAB-Neustarts, kein Whitelist-Pflegeaufwand.

Das Exec Binding bleibt dann für das, wofür es gedacht ist: **einmalige lokale Systemaktionen**.

---

## Checkliste

* [ ] Jeder Exec-Befehl steht **konkret** in `misc/exec.whitelist` – kein `sudo %2$s`, kein offener Platzhalter
* [ ] Keine Passwörter im Klartext (kein `sshpass -p`, kein `PsExec -p`)
* [ ] Remote-Zugriff per **SSH-Key**, Host-Keys in `known_hosts`
* [ ] `sudo` nur für einzeln benannte Befehle (`/etc/sudoers.d/…`, via `visudo`)
* [ ] Für Gerätesteuerung geprüft, ob **MQTT/Binding/eigener Dienst** besser passt als Exec
* [ ] Lange, fragile `grep … | awk …`-Ketten durch `systemctl show` oder Monitoring ersetzt

---
