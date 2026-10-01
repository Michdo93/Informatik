# 🤖 Backups mit Ansible

Ansible eignet sich auf zwei Arten für Backups:

1. **Backups ausrollen und ausführen:** Skripte, Cron-Jobs und NAS-Mounts werden auf allen Geräten einheitlich eingerichtet; Backups werden zentral angestoßen oder eingesammelt.
2. **Ansible selbst als Backup:** Wenn die komplette Konfiguration eines Geräts als Code beschrieben ist, lässt sich das Gerät jederzeit neu aufbauen. Gesichert werden muss dann nur noch der **Zustand** (Daten), nicht mehr die Installation.

Grundlagen und Struktur: [Best Practices → Ansible](../Best%20Practices/Ansible.md).

<!-- TOC -->
## Inhaltsverzeichnis

- [Backup-Rolle für alle Geräte](#backup-rolle-für-alle-geräte)
- [Backups zentral einsammeln](#backups-zentral-einsammeln)
- [Netzwerkgeräte sichern](#netzwerkgeräte-sichern)
- [Infrastructure as Code als Backup](#infrastructure-as-code-als-backup)
<!-- /TOC -->

## Backup-Rolle für alle Geräte

Eine Rolle `backup`, die auf jedem Gerät das gleiche Grundgerüst einrichtet, parametrisiert über Variablen:

`roles/backup/defaults/main.yml`:

```yaml
backup_nas_share: "//nas/backups"
backup_mount: /mnt/nas
backup_sources:
  - /etc
backup_keep_weekly: 5
backup_keep_monthly: 12
backup_keep_yearly: 5
```

`host_vars/pi-beamer.yml` – nur was abweicht:

```yaml
backup_sources:
  - /etc
  - /opt/beamer-control
  - /etc/mosquitto
```

`roles/backup/tasks/main.yml`:

```yaml
- name: Install required packages
  ansible.builtin.apt:
    name: [cifs-utils, zstd, rsync]
    state: present

- name: Store NAS credentials
  ansible.builtin.copy:
    dest: /root/.smb-nas
    content: |
      username={{ vault_backup_nas_user }}
      password={{ vault_backup_nas_password }}
    mode: "0600"
  no_log: true

- name: Mount NAS
  ansible.posix.mount:
    src: "{{ backup_nas_share }}"
    path: "{{ backup_mount }}"
    fstype: cifs
    opts: "credentials=/root/.smb-nas,_netdev,nofail,x-systemd.automount"
    state: mounted

- name: Install backup library and scripts
  ansible.builtin.template:
    src: "{{ item }}.j2"
    dest: "/usr/local/bin/{{ item }}"
    mode: "0755"
  loop:
    - backup_weekly.sh
    - backup_monthly.sh
    - backup_yearly.sh

- name: Schedule backups
  ansible.builtin.cron:
    name: "{{ item.name }}"
    minute: "{{ item.minute }}"
    hour: "{{ item.hour }}"
    day: "{{ item.day | default('*') }}"
    month: "{{ item.month | default('*') }}"
    weekday: "{{ item.weekday | default('*') }}"
    job: "/usr/local/bin/{{ item.name }}.sh >> /var/log/{{ item.name }}.log 2>&1"
  loop:
    - { name: backup_weekly,  minute: 30, hour: 2, weekday: 0 }
    - { name: backup_monthly, minute: 30, hour: 3, day: 1 }
    - { name: backup_yearly,  minute: 30, hour: 4, day: 1, month: 1 }
```

Die Skripte selbst sind in [Backup-Skripte & Intervalle](Backup-Skripte%20&%20Intervalle.md) beschrieben; als Jinja2-Template erhalten sie Quellen und Retention aus den Variablen.

---

## Backups zentral einsammeln

Statt dass jedes Gerät selbst auf das NAS schreibt, kann ein Playbook die Daten **holen**. Vorteil: Die Geräte brauchen keinen Schreibzugriff auf das NAS (Ransomware auf einem Gerät kann dann die Backups nicht verschlüsseln).

`playbooks/backup.yml`:

```yaml
- name: Collect configuration backups
  hosts: all
  become: true
  vars:
    stamp: "{{ ansible_date_time.date }}"
    local_dest: "/srv/backups/{{ inventory_hostname }}"
  tasks:
    - name: Create archive on the target
      community.general.archive:
        path: "{{ backup_sources }}"
        dest: "/tmp/{{ inventory_hostname }}_{{ stamp }}.tar.gz"
        format: gz
        mode: "0600"

    - name: Fetch archive to the control node
      ansible.builtin.fetch:
        src: "/tmp/{{ inventory_hostname }}_{{ stamp }}.tar.gz"
        dest: "{{ local_dest }}/"
        flat: true

    - name: Remove temporary archive
      ansible.builtin.file:
        path: "/tmp/{{ inventory_hostname }}_{{ stamp }}.tar.gz"
        state: absent

- name: Dump databases
  hosts: db_servers
  become: true
  tasks:
    - name: Dump all MariaDB databases
      ansible.builtin.shell: >
        mysqldump --single-transaction --all-databases
        | zstd > /tmp/mariadb_{{ ansible_date_time.date }}.sql.zst
      args:
        executable: /bin/bash
      changed_when: true

    - name: Fetch dump
      ansible.builtin.fetch:
        src: "/tmp/mariadb_{{ ansible_date_time.date }}.sql.zst"
        dest: "/srv/backups/{{ inventory_hostname }}/"
        flat: true
```

Ausführen – manuell oder per Cron auf dem Control Node:

```cron
0 1 * * * cd /opt/ansible && ansible-playbook playbooks/backup.yml >> /var/log/ansible_backup.log 2>&1
```

---

## Netzwerkgeräte sichern

Für Switches, Router und Firewalls gibt es herstellerspezifische Module, die die laufende Konfiguration sichern:

```yaml
- name: Backup switch configuration
  hosts: switches
  gather_facts: false
  tasks:
    - name: Save running config
      cisco.ios.ios_config:
        backup: true
        backup_options:
          dir_path: /srv/backups/network
          filename: "{{ inventory_hostname }}_{{ lookup('pipe', 'date +%F') }}.cfg"
```

---

## Infrastructure as Code als Backup

Wenn ein Raspberry Pi stirbt, gibt es zwei Wege zurück:

| Weg | Ablauf | Problem |
| --- | --- | --- |
| **Image-Backup** | SD-Karten-Image zurückspielen | Image ist groß, alt, enthält Unbekanntes, passt evtl. nicht auf neue Hardware |
| **Ansible** | Neues Betriebssystem flashen → [DHCP](../Best%20Practices/DHCP.md)-Reservierung auf neue MAC → `ansible-playbook site.yml --limit pi-beamer` → Daten-Backup zurückspielen | Funktioniert nur, wenn **alles** im Playbook steht |

Der zweite Weg ist sauberer, nachvollziehbar und funktioniert auch für das „gleiche Gerät nochmal“ (z. B. zweiter Beamer-Pi für ein anderes Labor). Deshalb gilt: **Was nicht in Ansible steht, ist nicht gesichert.**

Das **Ansible-Repository selbst** muss dann natürlich besonders gut gesichert sein (Git mit Remote, siehe [Git als Backup](Git%20als%20Backup.md)), inklusive des Vault-Passworts an einem sicheren, separaten Ort.

---
