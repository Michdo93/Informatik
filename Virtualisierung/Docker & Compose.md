# 🐳 Docker & Compose betreiben

Container machen Dienste leicht installierbar – aber nur, wenn sie **sauber konfiguriert** sind. Ein mit `docker run` gestarteter Container, dessen Optionen niemand mehr kennt, ist nach einem Update oder Neustart des Hosts schnell verloren. Dieses Kapitel beschreibt, wie wir Container im Labor betreiben.

<!-- TOC -->
## Inhaltsverzeichnis

- [Begriffe](#begriffe)
- [Von docker run zu Compose](#von-docker-run-zu-compose)
- [Struktur auf dem Docker-Host](#struktur-auf-dem-docker-host)
- [Gute Konfiguration](#gute-konfiguration)
- [Betrieb](#betrieb)
  - [Updates](#updates)
- [Backups](#backups)
- [Docker in VM oder LXC?](#docker-in-vm-oder-lxc)
- [Checkliste pro Container](#checkliste-pro-container)
<!-- /TOC -->

## Begriffe

| Begriff | Bedeutung |
| --- | --- |
| **Image** | Unveränderliche Vorlage (Dateisystem + Startbefehl), z. B. `eclipse-mosquitto:2.0.20` |
| **Container** | Laufende Instanz eines Images |
| **Tag** | Version eines Images (`2.0.20`, `latest`) |
| **Volume** | Persistenter Speicher außerhalb des Containers |
| **Bind-Mount** | Verzeichnis des Hosts im Container (`./config:/mosquitto/config`) |
| **Netzwerk** | Virtuelles Netz, in dem Container sich per Namen erreichen |
| **Compose** | Beschreibung mehrerer Container als YAML-Datei (`compose.yaml`) |
| **Stack** | Zusammengehörige Container einer Compose-Datei |
| **Registry** | Ablage für Images (Docker Hub, GitHub Container Registry, eigene) |

> **Wichtigste Regel:** Ein Container ist **wegwerfbar**. Alles, was nicht verloren gehen darf (Daten, Konfiguration), gehört in ein **Volume** oder einen **Bind-Mount**.

---

## Von `docker run` zu Compose

```bash
# hard to reproduce: nobody remembers these options a year later
docker run -d --name mosquitto -p 8883:8883 -v /srv/mqtt:/mosquitto eclipse-mosquitto
```

```yaml
# /opt/stacks/mosquitto/compose.yaml - versioned, documented, reproducible
services:
  mosquitto:
    image: eclipse-mosquitto:2.0.20          # fixed version instead of :latest
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "8883:8883"
    volumes:
      - ./config:/mosquitto/config:ro       # configuration: bind mount, read-only
      - mosquitto-data:/mosquitto/data       # data: named volume
      - ./log:/mosquitto/log
    logging:
      driver: json-file
      options:
        max-size: "10m"                      # limit log size
        max-file: "3"

volumes:
  mosquitto-data:
```

**Bestehende Container übernehmen:** Mit `docker inspect <container>` lassen sich Image, Ports, Volumes, Umgebungsvariablen und Netzwerke auslesen und in eine `compose.yaml` übertragen. Danach den alten Container stoppen (nicht sofort löschen), den Stack mit `docker compose up -d` starten und testen.

---

## Struktur auf dem Docker-Host

```text
/opt/stacks/
├── mosquitto/
│   ├── compose.yaml
│   ├── .env               # secrets, chmod 600, NOT in git
│   ├── .env.example       # template without secrets, in git
│   └── config/
├── influxdb/
│   └── compose.yaml
└── grafana/
    └── compose.yaml
```

* **Ein Verzeichnis pro Stack**, Name = Zweck.
* Compose-Dateien in einem **Git-Repository** versionieren (ohne `.env`).
* Eine kurze `README.md` pro Stack: Zweck, URL, Ports, Besonderheiten, Update-Hinweise.

---

## Gute Konfiguration

| Thema | Empfehlung |
| --- | --- |
| **Versionen** | Feste Tags (`2.0.20`), nicht `latest` – sonst ändert sich beim nächsten `pull` unbemerkt die Version |
| **Neustart** | `restart: unless-stopped` |
| **Geheimnisse** | In `.env` (Rechte `600`) oder Docker Secrets, nie in `compose.yaml` |
| **Ports** | Nur veröffentlichen, was nötig ist; Weboberflächen nur auf `127.0.0.1` hinter einem Reverse Proxy (`"127.0.0.1:3000:3000"`) |
| **Netzwerke** | Container, die miteinander sprechen, in ein gemeinsames Netzwerk; Datenbanken **nicht** nach außen veröffentlichen |
| **Healthcheck** | Damit Docker (und abhängige Container) erkennen, ob ein Dienst wirklich bereit ist |
| **Logs** | Größe begrenzen (`max-size`), sonst läuft die Platte voll |
| **Benutzer** | Wenn das Image es unterstützt: nicht als root (`user: "1000:1000"`) |
| **Ressourcen** | Bei Bedarf Grenzen setzen (`mem_limit`, `cpus`) |
| **Zeitzone** | `TZ=Europe/Berlin` setzen, wenn Zeitstempel wichtig sind |

```yaml
services:
  app:
    image: example/app:1.4.2
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}      # from .env
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      retries: 5
volumes:
  db-data:
```

---

## Betrieb

```bash
cd /opt/stacks/mosquitto
docker compose up -d                 # create/update and start
docker compose ps                    # status
docker compose logs -f --tail 100    # logs
docker compose pull                  # fetch new images (after changing the tag)
docker compose up -d                 # recreate changed containers
docker compose down                  # stop and remove containers (volumes stay!)

docker ps -a                         # all containers on the host
docker system df                     # disk usage
docker image prune                   # remove unused images
```

> ⚠️ `docker compose down -v` und `docker volume prune` **löschen Volumes** – und damit Daten. Nur verwenden, wenn man genau weiß, was man tut.

### Updates

1. Release Notes des neuen Image-Tags lesen (Breaking Changes, Migrationen).
2. **Backup** der Volumes.
3. Tag in `compose.yaml` erhöhen, committen.
4. `docker compose pull && docker compose up -d`.
5. Logs und Funktion prüfen; bei Problemen alten Tag wiederherstellen.

Automatische Updates (z. B. mit Watchtower) sind bequem, aber riskant: Ein fehlerhaftes Image kann einen Dienst unbemerkt lahmlegen. Im Labor lieber **bewusst und dokumentiert** aktualisieren.

---

## Backups

| Was | Wie |
| --- | --- |
| Compose-Dateien und Konfiguration | Git-Repository (→ [Git als Backup](../Backup-Strategien/Git%20als%20Backup.md)) |
| Bind-Mounts | Mit dem Host sichern (tar, rsync) |
| Benannte Volumes | Liegen unter `/var/lib/docker/volumes/`; konsistent sichern bei **gestopptem** Container oder per Export |
| Datenbanken | Mit dem passenden Dump-Werkzeug (`pg_dump`, `mysqldump`, `influx backup`), nicht als rohe Dateien im laufenden Betrieb |
| Ganze Docker-VM | Proxmox-Backup als zusätzliche Absicherung (→ [Proxmox](Proxmox.md)) |

```bash
# export a named volume as tar archive (container stopped)
docker compose stop
docker run --rm -v mosquitto_mosquitto-data:/data -v "$PWD":/backup alpine \
    tar czf /backup/mosquitto-data_$(date +%F).tar.gz -C /data .
docker compose start
```

(Der Volume-Name setzt sich aus Projektname und Volume-Name zusammen – mit `docker volume ls` prüfen.)

---

## Docker in VM oder LXC?

Docker lässt sich in einem Proxmox-LXC betreiben (`nesting=1`, `keyctl=1`), es gibt aber immer wieder Probleme mit Dateisystemen, Berechtigungen und Updates. Für einen Docker-Host mit vielen Containern ist eine **VM** die robustere Wahl. Einzelne einfache Dienste lassen sich dagegen oft direkt als **LXC** ohne Docker betreiben (→ [Proxmox](Proxmox.md)).

---

## Checkliste pro Container

* [ ] Läuft über eine `compose.yaml` im Stack-Verzeichnis
* [ ] Image mit fester Version
* [ ] Persistente Daten in Volume/Bind-Mount – geprüft, dass nichts im Container-Dateisystem liegt
* [ ] Geheimnisse in `.env` (Rechte `600`), `.env.example` vorhanden
* [ ] `restart: unless-stopped`, Log-Begrenzung, ggf. Healthcheck
* [ ] Nur nötige Ports veröffentlicht; Weboberfläche über Reverse Proxy mit HTTPS
* [ ] Im Backup enthalten, Wiederherstellung getestet
* [ ] README im Stack-Verzeichnis

---
