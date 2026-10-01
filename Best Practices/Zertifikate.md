# 🔐 Best Practice: Zertifikate & TLS

**Regel:** Ein Dienst bekommt **von Anfang an** ein Zertifikat und wird über eine verschlüsselte Verbindung (HTTPS, MQTTS, …) betrieben. Zertifikate werden **automatisch und regelmäßig erneuert**.

<!-- TOC -->
## Inhaltsverzeichnis

- [Warum von Anfang an?](#warum-von-anfang-an)
- [Begriffe](#begriffe)
- [Welche Art von Zertifikat?](#welche-art-von-zertifikat)
- [Eigene CA mit OpenSSL](#eigene-ca-mit-openssl)
- [Dateinamen-Konvention](#dateinamen-konvention)
- [Regelmäßig erneuern](#regelmäßig-erneuern)
- [Sicherheitsregeln](#sicherheitsregeln)
- [Checkliste](#checkliste)
<!-- /TOC -->

## Warum von Anfang an?

In den ersten Schritten läuft ein Dienst oft „nur mal schnell“ über HTTP ohne Zertifikat. Das Problem:

> **Nichts ist so beständig wie ein Provisorium.**

Wenn TLS nicht beim Aufsetzen eingerichtet wird, passiert es später meist nie. Andere Systeme werden inzwischen gegen `http://...:8080` konfiguriert, Clients ohne TLS-Unterstützung gebaut, Firewall-Regeln für den unverschlüsselten Port angelegt. Ein späterer Umstieg betrifft dann viele Stellen gleichzeitig – und wird deshalb aufgeschoben.

Wer dagegen **beim ersten Start** ein Zertifikat erstellt, hat:

* Passwörter, Tokens und Daten nie im Klartext im Netz,
* alle Clients von Anfang an korrekt konfiguriert (inkl. CA-Prüfung),
* keine „später“-Aufgabe auf der Liste.

Das gilt genauso für **MQTT** (Port 8883 statt 1883), **Datenbanken**, **REST-APIs**, **WebSockets** (`wss://`) und **LDAP** (`ldaps://`).

---

## Begriffe

| Begriff | Bedeutung |
| --- | --- |
| **TLS** | *Transport Layer Security*, Nachfolger von SSL. Verschlüsselt und authentifiziert Verbindungen. „SSL-Zertifikat“ ist umgangssprachlich, technisch ist TLS gemeint. |
| **Zertifikat (X.509)** | Öffentlicher Schlüssel + Identität (Hostname/IP) + Gültigkeitszeitraum, unterschrieben von einer CA. |
| **Privater Schlüssel** | Geheimer Gegenpart zum Zertifikat. Verlässt niemals den Server. |
| **CA** | *Certificate Authority*. Stelle, die Zertifikate signiert. Öffentlich (z. B. Let's Encrypt) oder selbst betrieben. |
| **Root-CA / Intermediate-CA** | Die Root-CA signiert Intermediate-CAs, diese signieren Server-Zertifikate (Vertrauenskette). |
| **CSR** | *Certificate Signing Request*. Antrag auf ein Zertifikat, enthält den öffentlichen Schlüssel. |
| **SAN** | *Subject Alternative Name*. Liste der Hostnamen/IPs, für die das Zertifikat gilt. **Moderne Clients prüfen nur noch den SAN**, nicht mehr den CN. |
| **Self-signed** | Zertifikat, das sich selbst unterschreibt. Muss bei jedem Client einzeln als vertrauenswürdig hinterlegt werden. |
| **mTLS** | *Mutual TLS*. Auch der Client weist sich mit einem Zertifikat aus. |

---

## Welche Art von Zertifikat?

| Situation | Empfehlung |
| --- | --- |
| Dienst ist öffentlich unter einer echten Domain erreichbar | **Let's Encrypt** mit `certbot` oder dem eingebauten ACME-Client des Reverse-Proxys (Caddy, Traefik) |
| Internes Netz / Labor / Smart Home | **Eigene CA** (eine Root-CA, mit der alle internen Zertifikate signiert werden). Die CA wird einmal auf allen Clients installiert. |
| Interne Domain, aber DNS beim Provider steuerbar | Let's Encrypt mit **DNS-Challenge** – funktioniert auch ohne öffentlich erreichbaren Server |
| Schneller Test auf dem eigenen Rechner | `mkcert` (erstellt lokale CA und Zertifikate automatisch) |
| Viele Dienste, automatische Erneuerung intern | `step-ca` (Smallstep) als interne ACME-CA |

Einzelne self-signed Zertifikate pro Dienst sind die schlechteste Variante: Jeder Client muss jedem Zertifikat einzeln vertrauen, oder – was in der Praxis passiert – die Prüfung wird abgeschaltet (`verify=False`, `--insecure`). Dann ist die Verschlüsselung gegen Man-in-the-Middle wertlos.

---

## Eigene CA mit OpenSSL

**1. CA erstellen (einmalig, sicher aufbewahren!)**

```bash
mkdir -p ~/pki && cd ~/pki
openssl genrsa -out lab_ca.key 4096
chmod 600 lab_ca.key
openssl req -x509 -new -key lab_ca.key -sha256 -days 3650 \
  -subj "/C=DE/O=Smart Home Lab/CN=Smart Home Lab Root CA" \
  -out lab_ca.crt
```

**2. Server-Zertifikat für einen Host erstellen**

```bash
HOST=pi-beamer
IP=192.168.1.51

openssl genrsa -out ${HOST}.key 2048
openssl req -new -key ${HOST}.key -subj "/CN=${HOST}" -out ${HOST}.csr

cat > ${HOST}.ext <<EOF
basicConstraints=CA:FALSE
keyUsage=digitalSignature,keyEncipherment
extendedKeyUsage=serverAuth
subjectAltName=DNS:${HOST},DNS:${HOST}.lab.local,IP:${IP}
EOF

openssl x509 -req -in ${HOST}.csr -CA lab_ca.crt -CAkey lab_ca.key -CAcreateserial \
  -days 397 -sha256 -extfile ${HOST}.ext -out ${HOST}.crt
```

**3. Prüfen**

```bash
openssl x509 -in ${HOST}.crt -noout -subject -enddate -ext subjectAltName
openssl verify -CAfile lab_ca.crt ${HOST}.crt
```

> Die Laufzeit von 397 Tagen entspricht der Obergrenze, die Browser für öffentliche Zertifikate akzeptieren. Kürzere Laufzeiten sind sicherer, setzen aber eine automatische Erneuerung voraus.

---

## Dateinamen-Konvention

Werden Zertifikate auf andere Rechner kopiert, liegen dort schnell mehrere `ca.crt`, `server.crt` und `server.key` – und niemand weiß mehr, welche zu welchem System gehört. Deshalb **immer den Hostnamen im Dateinamen**:

| Datei | Inhalt |
| --- | --- |
| `<hostname>_ca.crt` | CA-Zertifikat, mit dem der Host signiert wurde (bzw. das der Host als Vertrauensanker nutzt) |
| `<hostname>.crt` | Server-Zertifikat |
| `<hostname>.key` | Privater Schlüssel (nur auf diesem Host!) |
| `<hostname>_fullchain.crt` | Server-Zertifikat + Intermediate-CA(s) |
| `<client>_client.crt` / `.key` | Client-Zertifikat bei mTLS |

Beispiel: Auf dem openHAB-Server liegt `pi-beamer_ca.crt`, damit openHAB dem MQTT-Broker auf `pi-beamer` vertraut. Man sieht sofort, wofür die Datei da ist.

**Ablageorte unter Linux:**

```text
/etc/ssl/certs/        # Zertifikate (öffentlich)
/etc/ssl/private/      # private Schlüssel (Rechte 700, Dateien 600)
/etc/mosquitto/certs/  # dienstspezifisch, z. B. für Mosquitto
```

Eigene CA systemweit vertrauen (Debian/Ubuntu):

```bash
sudo cp lab_ca.crt /usr/local/share/ca-certificates/lab_ca.crt
sudo update-ca-certificates
```

---

## Regelmäßig erneuern

Zertifikate laufen ab – und zwar immer im ungünstigsten Moment. Deshalb:

1. **Automatisch erneuern.** `certbot` installiert selbst einen `systemd`-Timer. Bei eigener CA: Erneuerungsskript per [Cron](../Linux%20&%20Werkzeuge/Cron%20&%20systemd-Timer.md) oder [Ansible](Ansible.md).
2. **Dienste nach der Erneuerung neu laden** (`systemctl reload nginx`, `systemctl restart mosquitto`) – z. B. über certbots `--deploy-hook`.
3. **Ablauf überwachen**, z. B. mit einem kleinen Check-Skript:

```bash
#!/usr/bin/env bash
# Warnt, wenn ein Zertifikat in weniger als 30 Tagen abläuft.
CERT=${1:-/etc/ssl/certs/$(hostname).crt}
if ! openssl x509 -checkend $((30*24*3600)) -noout -in "$CERT"; then
  echo "WARNUNG: $CERT läuft in weniger als 30 Tagen ab!" >&2
  exit 1
fi
```

Für entfernte Dienste:

```bash
echo | openssl s_client -connect pi-beamer:8883 -servername pi-beamer 2>/dev/null \
  | openssl x509 -noout -enddate
```

---

## Sicherheitsregeln

* Private Schlüssel **niemals** in Git committen, per Mail verschicken oder in Chats posten. `.gitignore`: `*.key`, `*.pem`.
* Dateirechte: Schlüssel `600`, Besitzer ist der Dienst-User (z. B. `mosquitto`).
* Der private Schlüssel der **CA** gehört nicht auf einen Server, sondern offline gesichert (verschlüsselter USB-Stick, Passwort-Manager).
* Zertifikatsprüfung **niemals** dauerhaft abschalten (`verify=False`, `curl -k`, `tls_insecure_set(True)`).
* Ein Zertifikat pro Host bzw. Dienst, kein Wildcard-Zertifikat mit Schlüssel auf zwanzig Maschinen.

---

## Checkliste

- [ ] Dienst läuft ab dem ersten Start mit TLS
- [ ] Zertifikat von der eigenen CA oder Let's Encrypt, mit korrektem SAN (Hostname **und** IP)
- [ ] Dateinamen nach Konvention (`<hostname>_ca.crt`, `<hostname>.crt`, `<hostname>.key`)
- [ ] Rechte des privaten Schlüssels `600`
- [ ] Automatische Erneuerung eingerichtet und getestet
- [ ] Ablaufüberwachung aktiv
- [ ] Unverschlüsselter Port deaktiviert oder nur noch auf `localhost`

---
