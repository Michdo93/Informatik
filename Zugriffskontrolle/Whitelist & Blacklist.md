# 📋 Whitelist & Blacklist

Eine **Whitelist** (Positivliste) zählt auf, was **erlaubt** ist – alles andere ist verboten. Eine **Blacklist** (Negativliste) zählt auf, was **verboten** ist – alles andere ist erlaubt.

<!-- TOC -->
## Inhaltsverzeichnis

- [Das Prinzip](#das-prinzip)
- [Begriffe und Namensherkunft](#begriffe-und-namensherkunft)
- [Beispiele aus verschiedenen Domänen](#beispiele-aus-verschiedenen-domänen)
  - [Foren und Communities](#foren-und-communities)
  - [Betriebssystem](#betriebssystem)
  - [Firewall](#firewall)
  - [openHAB Exec-Binding](#openhab-exec-binding)
  - [E-Mail](#e-mail)
  - [Netzwerk und Web](#netzwerk-und-web)
  - [Programmierung](#programmierung)
- [Kombination](#kombination)
<!-- /TOC -->

## Das Prinzip

| | Whitelist (Allowlist) | Blacklist (Denylist / Blocklist) |
| --- | --- | --- |
| Grundhaltung | **Default Deny** – alles verboten, Ausnahmen erlaubt | **Default Allow** – alles erlaubt, Ausnahmen verboten |
| Sicherheit | hoch: Unbekanntes wird blockiert | niedriger: Unbekanntes kommt durch |
| Pflegeaufwand | Jede neue legitime Sache muss eingetragen werden | Jede neue Bedrohung muss eingetragen werden |
| Fehlerbild | Legitimes wird blockiert („geht nicht“) | Schädliches kommt durch („wurde gehackt“) |
| Typischer Einsatz | Firewalls (eingehend), Befehlsausführung, Admin-Zugänge | Spam-Filter, Foren-Wortfilter, Werbeblocker, gesperrte Benutzer |

**Faustregel für Sicherheit: Whitelist, wann immer die Menge des Erlaubten überschaubar ist.** Die Menge möglicher Angriffe ist unendlich, die Menge legitimer Befehle meist nicht.

---

## Begriffe und Namensherkunft

Die Begriffe stammen aus dem Englischen: Eine *black list* war im 17. Jahrhundert eine Liste von Personen, die bestraft oder gemieden werden sollten; *white list* entstand später als Gegenstück. Viele Projekte und Firmen (Google, GitHub, Linux-Kernel, Mosquitto u. a.) verwenden heute bevorzugt die neutraleren und zugleich präziseren Begriffe:

| Alt | Neu |
| --- | --- |
| Whitelist | **Allowlist** |
| Blacklist | **Denylist**, **Blocklist** |
| Greylist | (unverändert, siehe unten) |

In älterer Software und Dokumentation (z. B. openHAB `exec.whitelist`) findet man weiterhin die alten Begriffe. Beide sind gleichbedeutend.

**Greylisting** ist eine Zwischenform aus der E-Mail-Welt: Unbekannte Absender werden beim ersten Zustellversuch vorübergehend abgewiesen. Echte Mailserver versuchen es später erneut, viele Spam-Programme nicht.

---

## Beispiele aus verschiedenen Domänen

### Foren und Communities

| Liste | Beispiel |
| --- | --- |
| Blacklist | Gesperrte Benutzer, gesperrte IP-Adressen, verbotene Wörter (Wortfilter), gesperrte Link-Domains |
| Whitelist | Erlaubte HTML-Tags in Beiträgen (`<b>`, `<i>`, `<a>` – aber kein `<script>`), freigeschaltete Benutzer in moderierten Foren, erlaubte Datei-Endungen bei Uploads |

Am Beispiel Forum sieht man gut, warum man für Sicherheit Whitelists nimmt: Wer HTML-Tags per Blacklist filtert (`<script>` verboten), vergisst garantiert eine Variante (`<img onerror=...>`, `<svg onload=...>`). Wer nur `<b>`, `<i>` und `<a href>` erlaubt, ist auf der sicheren Seite.

### Betriebssystem

| Liste | Beispiel |
| --- | --- |
| Whitelist | `/etc/sudoers.d/`: Ein Dienst-Account darf genau **einen** Befehl mit Root-Rechten ausführen |
| Whitelist | `AllowUsers admin ansible` in `sshd_config` |
| Whitelist | AppArmor/SELinux-Profile: Ein Programm darf nur bestimmte Dateien öffnen |
| Blacklist | `/etc/modprobe.d/blacklist.conf`: Kernelmodule, die nicht geladen werden sollen |
| Blacklist | `DenyUsers` in `sshd_config`, `/etc/hosts.deny` (veraltet) |

```text
# /etc/sudoers.d/openhab  – Whitelist für genau einen Befehl
openhab ALL=(root) NOPASSWD: /usr/bin/systemctl restart mosquitto
```

### Firewall

Firewalls arbeiten mit einer **Default-Policy** und Regeln. Die sichere Grundeinstellung ist **eingehend Whitelist** (alles verboten, gezielt erlauben):

```bash
sudo ufw default deny incoming     # Whitelist-Prinzip für eingehende Verbindungen
sudo ufw default allow outgoing
sudo ufw allow from 192.168.1.0/24 to any port 2222 proto tcp   # SSH nur aus dem Labornetz
sudo ufw allow 8883/tcp            # MQTT über TLS
sudo ufw deny from 203.0.113.17    # Blacklist-Eintrag für eine einzelne IP
sudo ufw enable
```

Werkzeuge wie **Fail2ban** pflegen automatisch eine **dynamische Blacklist**: IPs mit zu vielen Fehlversuchen werden für eine Zeit gesperrt.

### openHAB Exec-Binding

Das Exec-Binding führt Kommandozeilenbefehle aus. Aus Sicherheitsgründen werden **nur Befehle ausgeführt, die in der Whitelist stehen** – alle anderen werden ignoriert (mit einer Warnung im Log). Die Datei liegt unter `$OPENHAB_CONF/misc/exec.whitelist`, bei Paketinstallationen also `/etc/openhab/misc/exec.whitelist`.

```text
# /etc/openhab/misc/exec.whitelist
# Jede Zeile ist ein exakt so erlaubter Befehl (inkl. Parameter-Platzhalter %2$s)
/usr/local/bin/beamer_power.sh %2$s
sshpass -f /etc/openhab/secrets/nas.pass ssh -p 2222 admin@192.168.1.20 sudo /sbin/poweroff
```

Der Befehl im Thing muss **exakt** mit einer Zeile übereinstimmen. Häufigster Fehler: zusätzliche Leerzeichen oder ein anderer Pfad.

### E-Mail

* **Blacklists** (DNSBL/RBL, z. B. Spamhaus): Listen bekannter Spam-IP-Adressen, die Mailserver abfragen.
* **Whitelists**: Bekannte Absender, die nie im Spam landen.
* **Greylisting** (s. o.).

### Netzwerk und Web

| Beispiel | Art |
| --- | --- |
| MAC-Filter im WLAN | Whitelist (leicht zu umgehen, da MACs fälschbar sind) |
| DNS-Blocker (Pi-hole, AdGuard) | Blacklist von Werbe-/Tracking-Domains |
| CORS (`Access-Control-Allow-Origin`) | Whitelist erlaubter Ursprünge für Browser-Anfragen |
| Uploads: erlaubte Dateitypen | Whitelist (`.png`, `.jpg`, `.pdf`) |
| Eingabevalidierung | Whitelist erlaubter Zeichen statt Blacklist gefährlicher Zeichen |

### Programmierung

* **Eingaben validieren per Whitelist:** Ein Gerätename darf nur `[a-z0-9-]` enthalten – statt zu versuchen, alle gefährlichen Zeichen (`;`, `|`, `&`, `` ` ``, `$(`) aufzuzählen.
* **Feature Flags**: Whitelists von Benutzern, die eine neue Funktion schon sehen dürfen.

---

## Kombination

In der Praxis werden beide Listen kombiniert. Dann ist die **Reihenfolge der Auswertung** entscheidend: Bei den meisten Firewalls gilt **die erste passende Regel** (*first match*), bei anderen Systemen gewinnt ein Verbot immer (*deny overrides*). Das muss man für das jeweilige System nachlesen.

Beispiel Forum: Whitelist für erlaubte HTML-Tags **und** Blacklist für gesperrte Link-Domains **und** Blacklist für gesperrte Benutzer.

---
