# 🤖 Best Practice: Ansible

[Ansible](https://docs.ansible.com/) ist ein Werkzeug zur **Konfigurationsverwaltung und Automatisierung**. Es verbindet sich per SSH mit den Zielrechnern und bringt sie in einen beschriebenen Soll-Zustand. Es wird kein Agent auf den Zielrechnern benötigt – nur SSH und Python.

**Regeln in Kurzform:**

1. Jedes neue Gerät kommt **sofort nach DHCP und SSH ins Ansible-Inventar**.
2. Geräte werden **gruppiert und klassifiziert**.
3. Aufgaben werden in **kleine, getrennte Rollen** zerlegt – lieber fünf wiederverwendbare Rollen als eine große Spezialrolle.
4. Geheimnisse liegen verschlüsselt in **Ansible Vault**.
5. Wiederkehrende Aufgaben (Cron-Jobs) werden **über Ansible** angelegt, nicht von Hand.

<!-- TOC -->
## Inhaltsverzeichnis

- [Begriffe](#begriffe)
- [Das Gerät gehört sofort ins Inventar](#das-gerät-gehört-sofort-ins-inventar)
- [Geräte gruppieren und klassifizieren](#geräte-gruppieren-und-klassifizieren)
- [Rollen trennen statt eine Monster-Rolle](#rollen-trennen-statt-eine-monster-rolle)
- [Verzeichnisstruktur](#verzeichnisstruktur)
- [Cron-Jobs mit Ansible](#cron-jobs-mit-ansible)
- [Weitere Regeln](#weitere-regeln)
- [Checkliste für ein neues Gerät](#checkliste-für-ein-neues-gerät)
<!-- /TOC -->

## Begriffe

| Begriff | Bedeutung |
| --- | --- |
| **Control Node** | Rechner, auf dem Ansible läuft |
| **Managed Node** | Zielrechner, der konfiguriert wird |
| **Inventory** | Liste der Zielrechner, gruppiert |
| **Module** | Einzelne Funktion, z. B. `apt`, `copy`, `template`, `service`, `cron` |
| **Task** | Ein Aufruf eines Moduls („installiere Paket X“) |
| **Handler** | Task, der nur bei Änderung ausgelöst wird („starte Dienst neu, wenn die Konfiguration geändert wurde“) |
| **Role** | Wiederverwendbares Paket aus Tasks, Templates, Variablen, Handlern |
| **Playbook** | Ordnet Rollen/Tasks Gruppen von Hosts zu |
| **Idempotenz** | Ein Playbook kann beliebig oft laufen; ist der Soll-Zustand erreicht, ändert sich nichts mehr |

---

## Das Gerät gehört sofort ins Inventar

Der Einrichtungsablauf für jedes neue Gerät:

```mermaid
flowchart LR
    A[Gerät auspacken] --> B[MAC ermitteln<br/>feste IP per DHCP]
    B --> C[SSH einrichten<br/>Port, Key, sshpass]
    C --> D[Ins Ansible-Inventar<br/>Gruppen zuordnen]
    D --> E[Playbook ausführen<br/>Basis-Rollen]
    E --> F[Dokumentation]
```

Siehe [DHCP](DHCP.md) und [SSH](SSH.md). Wer ein Gerät „nur kurz von Hand“ einrichtet, hat später ein Gerät, dessen Zustand niemand kennt und das niemand reproduzieren kann. Ab dem Eintrag ins Inventar ist der Ansible-Code die Dokumentation des Gerätezustands.

Für die **Erst-Einrichtung**, bevor der SSH-Schlüssel verteilt ist, kann Ansible per Passwort verbinden (dafür braucht es `sshpass` auf dem Control Node):

```bash
ansible-playbook -i inventory/hosts.yml bootstrap.yml --limit pi-beamer --ask-pass --ask-become-pass
```

---

## Geräte gruppieren und klassifizieren

Ein Gerät kann in **mehreren Gruppen** gleichzeitig sein. Gruppiert wird nach unterschiedlichen Merkmalen:

| Merkmal | Beispielgruppen |
| --- | --- |
| Funktion / Rolle | `mqtt_brokers`, `openhab_servers`, `cameras`, `beamers`, `ros_robots` |
| Hardware / Plattform | `raspberry_pi`, `proxmox_vms`, `lxc_containers`, `x86_servers` |
| Betriebssystem | `debian`, `ubuntu`, `raspios` |
| Standort | `lab_room_a`, `lab_room_b` |
| Umgebung | `production`, `testing`, `students` |

`inventory/hosts.yml`:

```yaml
all:
  vars:
    ansible_port: 2222
    ansible_user: ansible
  children:
    raspberry_pi:
      hosts:
        pi-beamer:   { ansible_host: 192.168.1.51 }
        pi-camera:   { ansible_host: 192.168.1.52 }
    proxmox_vms:
      hosts:
        vm-openhab:  { ansible_host: 192.168.1.30 }
    mqtt_brokers:
      hosts:
        pi-beamer:
        pi-camera:
    openhab_servers:
      hosts:
        vm-openhab:
```

Variablen pro Gruppe und pro Host:

```text
inventory/
├── hosts.yml
├── group_vars/
│   ├── all.yml              # gilt für alle
│   ├── raspberry_pi.yml     # z. B. Zeitzone, Swap-Größe
│   └── mqtt_brokers/
│       ├── main.yml
│       └── vault.yml        # verschlüsselte Passwörter
└── host_vars/
    └── pi-beamer.yml        # z. B. Gerätename, Topics
```

---

## Rollen trennen statt eine Monster-Rolle

Der wichtigste Grundsatz für wartbares Ansible: **Jede Rolle macht genau eine Sache.**

**Schlecht:** Eine Rolle `beamer_pi`, die Benutzer anlegt, SSH härtet, Mosquitto installiert, Zertifikate kopiert, das Python-Programm installiert und Cron-Jobs anlegt. Für die Kamera muss alles kopiert und angepasst werden – zwei Rollen, die auseinanderlaufen.

**Gut:** Fünf kleine Rollen, von denen drei auch für jedes andere Gerät taugen:

| Rolle | Aufgabe | Wiederverwendbar für |
| --- | --- | --- |
| `base` | Pakete, Zeitzone, NTP, Benutzer | **alle** Geräte |
| `ssh_hardening` | Port, Keys, `sshpass`, Fail2ban | **alle** Geräte |
| `mosquitto` | Broker, TLS, Passwörter, ACL | **alle** Geräte mit eigenem Broker |
| `tls_certs` | Zertifikate verteilen | **alle** Geräte mit TLS |
| `beamer_control` | Steuerprogramm für den Beamer | nur Beamer |

```yaml
# site.yml
- hosts: all
  become: true
  roles:
    - base
    - ssh_hardening

- hosts: mqtt_brokers
  become: true
  roles:
    - tls_certs
    - mosquitto

- hosts: beamers
  become: true
  roles:
    - beamer_control
```

Vorteile:

* **Wiederverwendbarkeit:** Neues Gerät = neue Gruppenzugehörigkeit, kaum neuer Code.
* **Testbarkeit:** Kleine Rollen lassen sich einzeln testen (`--tags`, Molecule).
* **Fehler sind lokalisiert:** Ein Fehler in `mosquitto` betrifft nicht die SSH-Härtung.
* **Lesbarkeit:** Das Playbook liest sich wie eine Inhaltsangabe.

Rollen werden über **Variablen** parametrisiert, nicht kopiert. Gemeinsame Rollen können über `requirements.yml` aus einem eigenen Repository oder aus **Ansible Galaxy** eingebunden werden.

---

## Verzeichnisstruktur

```text
ansible/
├── ansible.cfg
├── inventory/
│   ├── hosts.yml
│   ├── group_vars/
│   └── host_vars/
├── roles/
│   ├── base/
│   │   ├── tasks/main.yml
│   │   ├── handlers/main.yml
│   │   ├── templates/
│   │   ├── files/
│   │   └── defaults/main.yml
│   ├── ssh_hardening/
│   └── mosquitto/
├── playbooks/
│   ├── bootstrap.yml
│   └── backup.yml
├── site.yml
└── requirements.yml
```

Eine neue Rolle anlegen (erzeugt das Grundgerüst, siehe [Scaffolding](../Software-Konzepte/Skeleton,%20Boilerplate%20&%20Scaffolding.md)):

```bash
ansible-galaxy role init roles/mosquitto
```

---

## Cron-Jobs mit Ansible

Wiederkehrende Aufgaben (Backups, Zertifikatserneuerung, Aufräumen) werden mit dem Modul `ansible.builtin.cron` angelegt. Begriffe: Der **Cron-Job** ist der einzelne Eintrag in der Crontab, die **Cron-Task** der Befehl, den er ausführt – im Alltag werden beide Begriffe synonym verwendet. Grundlagen zu Cron: [Cron & systemd-Timer](../Linux%20&%20Werkzeuge/Cron%20&%20systemd-Timer.md).

```yaml
- name: Install weekly backup script
  ansible.builtin.copy:
    src: backup_weekly.sh
    dest: /usr/local/bin/backup_weekly.sh
    mode: "0755"

- name: Schedule weekly backup (Sunday 02:30)
  ansible.builtin.cron:
    name: "weekly backup"          # eindeutiger Name, darüber erkennt Ansible den Eintrag wieder
    user: root
    weekday: "0"
    hour: "2"
    minute: "30"
    job: "/usr/local/bin/backup_weekly.sh >> /var/log/backup_weekly.log 2>&1"

- name: Remove an old cron job
  ansible.builtin.cron:
    name: "old cleanup"
    state: absent
```

Wichtig: Der Parameter `name` ist Pflicht für Idempotenz. Ohne ihn legt jeder Lauf einen weiteren Eintrag an. Ansible markiert seine Einträge in der Crontab mit `#Ansible: <name>`.

Siehe auch [Backup-Strategien → Backups mit Ansible](../Backup-Strategien/Backups%20mit%20Ansible.md).

---

## Weitere Regeln

* **Module statt `shell`/`command`.** `ansible.builtin.apt` ist idempotent, `shell: apt install ...` nicht.
* **Vollständige Modulnamen** (FQCN) verwenden: `ansible.builtin.copy` statt `copy`.
* **Jeder Task hat einen `name`.** Er ist die Dokumentation im Log.
* **Templates** (`.j2`, Jinja2) für Konfigurationsdateien statt `lineinfile`-Ketten – siehe [Templates & Templating](../Software-Konzepte/Templates%20&%20Templating.md).
* **Geheimnisse mit Vault:** `ansible-vault create inventory/group_vars/mqtt_brokers/vault.yml`, Variablen mit Präfix `vault_`.
* **Erst prüfen, dann ändern:** `ansible-playbook site.yml --check --diff`
* **Gezielt ausführen:** `--limit pi-beamer`, `--tags mosquitto`
* **Linting:** `ansible-lint`
* Das Ansible-Repository liegt in **Git**.

---

## Checkliste für ein neues Gerät

- [ ] Feste IP und Hostname ([DHCP](DHCP.md))
- [ ] SSH mit eigenem Port und Schlüssel ([SSH](SSH.md))
- [ ] Eintrag in `inventory/hosts.yml` mit allen passenden Gruppen
- [ ] `host_vars/<hostname>.yml` falls nötig
- [ ] `ansible <hostname> -m ping` erfolgreich
- [ ] `site.yml --limit <hostname>` erfolgreich, zweiter Lauf ohne Änderungen (Idempotenz)
- [ ] Cron-Jobs (z. B. Backup) über Ansible angelegt
- [ ] Commit ins Ansible-Repository

---
