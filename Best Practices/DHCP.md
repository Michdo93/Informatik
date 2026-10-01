# 🌐 Best Practice: DHCP & feste IP-Adressen

**Regel:** Sobald ein neues Gerät in Betrieb genommen wird, wird seine **MAC-Adresse ermittelt** und ihm **sofort eine feste IP-Adresse** zugewiesen. Es wird nicht mit dynamisch vergebenen Adressen weitergearbeitet. Ein Hostname ist sehr gut und sollte ebenfalls gesetzt werden – er **ersetzt aber nicht** die feste IP.

<!-- TOC -->
## Inhaltsverzeichnis

- [Warum feste IP-Adressen?](#warum-feste-ip-adressen)
- [Begriffe](#begriffe)
- [DHCP-Reservierung vs. statische IP](#dhcp-reservierung-vs-statische-ip)
- [Vorgehen bei einem neuen Gerät](#vorgehen-bei-einem-neuen-gerät)
- [MAC-Adresse herausfinden](#mac-adresse-herausfinden)
- [Adressplan](#adressplan)
- [Beispiele für Reservierungen](#beispiele-für-reservierungen)
- [Hostname setzen](#hostname-setzen)
- [Checkliste](#checkliste)
<!-- /TOC -->

## Warum feste IP-Adressen?

* **Automatisierung braucht stabile Ziele.** Ansible-Inventar, openHAB-Things, MQTT-Konfigurationen, Firewall-Regeln, Reverse-Proxys und Zertifikate verweisen auf Adressen. Ändert sich die IP nach einem Neustart, bricht alles.
* **Hostnamen sind nicht immer auflösbar.** mDNS (`.local`) funktioniert nicht über Subnetze, nicht in jedem Container und nicht auf jedem Gerät. Lokales DNS fällt manchmal aus. Die IP funktioniert immer.
* **Fehlersuche wird einfacher.** Man weiß, welches Gerät hinter `192.168.1.42` steckt, und kann Logs, Firewall-Einträge und Netzwerkmitschnitte zuordnen.
* **Provisorien werden dauerhaft.** „Ich trage die IP später fest ein“ passiert erfahrungsgemäß nie – bis das Gerät nach einem Stromausfall eine neue Adresse bekommt.

---

## Begriffe

| Begriff | Bedeutung |
| --- | --- |
| **DHCP** | *Dynamic Host Configuration Protocol*. Ein Server vergibt Geräten automatisch IP-Adresse, Netzmaske, Gateway und DNS-Server. |
| **Lease** | Die „Ausleihe“ einer IP für eine bestimmte Zeit. Danach kann die Adresse neu vergeben werden. |
| **DHCP-Pool / Range** | Adressbereich, aus dem der Server dynamisch vergibt. |
| **DHCP-Reservierung (Static Lease)** | Der DHCP-Server vergibt einer bestimmten MAC-Adresse **immer dieselbe IP**. |
| **Statische IP** | Die Adresse wird **auf dem Gerät selbst** fest eingetragen, ohne DHCP. |
| **MAC-Adresse** | Hardware-Adresse der Netzwerkschnittstelle, z. B. `b8:27:eb:12:34:56`. Die ersten drei Bytes (OUI) verraten den Hersteller. |

---

## DHCP-Reservierung vs. statische IP

| | DHCP-Reservierung (empfohlen) | Statische IP auf dem Gerät |
| --- | --- | --- |
| Konfiguration | zentral am DHCP-Server | auf jedem Gerät einzeln |
| Überblick | alle Zuordnungen an einer Stelle | verteilt, leicht vergessen |
| Gateway/DNS ändern | einmal zentral | auf jedem Gerät |
| Funktioniert ohne DHCP-Server | nein | ja |
| Typischer Einsatz | Clients, IoT-Geräte, Raspberry Pis, Drucker | Router, DHCP-/DNS-Server selbst, Hypervisor, Netzwerk-Switches |

**Empfehlung:** Standardmäßig **DHCP-Reservierung**. Statische IPs nur für Infrastruktur, die selbst Voraussetzung für DHCP ist. Statische Adressen müssen **außerhalb des DHCP-Pools** liegen, sonst drohen Adresskonflikte.

---

## Vorgehen bei einem neuen Gerät

1. **MAC-Adresse ermitteln** (siehe unten).
2. **IP gemäß Adressplan festlegen** und prüfen, dass sie frei ist (`ping`, `arp`, DHCP-Lease-Liste).
3. **Reservierung am DHCP-Server anlegen** (MAC → IP, Hostname).
4. **Gerät neu verbinden** bzw. Lease erneuern und prüfen, ob es die richtige IP hat.
5. **Hostname setzen** (auf dem Gerät und im DNS).
6. **Dokumentieren** (Inventar/IP-Liste: IP, MAC, Hostname, Standort, Zweck, Verantwortlicher).
7. Weiter mit [SSH](SSH.md), [Zertifikaten](Zertifikate.md) und dem Eintrag in [Ansible](Ansible.md).

---

## MAC-Adresse herausfinden

| Wo | Befehl / Ort |
| --- | --- |
| Linux (auf dem Gerät) | `ip link show` bzw. `ip -br link` |
| Windows | `ipconfig /all` oder `getmac` |
| macOS | `ifconfig en0 \| grep ether` |
| Im Netz (Gerät hat schon eine IP) | `ip neigh` bzw. `arp -a` nach einem `ping` |
| Netzwerk-Scan | `sudo nmap -sn 192.168.1.0/24` (zeigt MAC + Hersteller) |
| Router / DHCP-Server | Lease-Liste, z. B. `cat /var/lib/misc/dnsmasq.leases` |
| Gerät selbst | Aufkleber, Verpackung, Web-Oberfläche, App |

> ⚠️ Viele Smartphones und Laptops nutzen **randomisierte MAC-Adressen** („Private WLAN-Adresse“). Für Geräte mit fester IP muss das für das jeweilige WLAN deaktiviert werden, sonst greift die Reservierung nicht.

---

## Adressplan

Ein Adressplan verhindert Chaos. Beispiel für ein `/24`-Netz:

| Bereich | Verwendung |
| --- | --- |
| `.1` | Router / Gateway |
| `.2` – `.19` | Netzwerk-Infrastruktur (Switches, Access Points, DNS) – statisch |
| `.20` – `.49` | Server, Hypervisor, VMs, NAS |
| `.50` – `.99` | Smart-Home- und IoT-Geräte, Raspberry Pis |
| `.100` – `.199` | **DHCP-Pool** (dynamisch, für Gäste und Unbekanntes) |
| `.200` – `.249` | Arbeitsplätze, Laborrechner, Drucker |
| `.250` – `.254` | Reserve / Test |

Der Plan selbst wird versioniert (z. B. als Markdown-Tabelle oder direkt als Ansible-Inventar).

---

## Beispiele für Reservierungen

**dnsmasq** (`/etc/dnsmasq.d/hosts.conf`):

```ini
dhcp-range=192.168.1.100,192.168.1.199,12h
dhcp-host=b8:27:eb:12:34:56,pi-beamer,192.168.1.51
dhcp-host=00:11:32:aa:bb:cc,nas,192.168.1.20
```

**ISC Kea** (`kea-dhcp4.conf`, Ausschnitt):

```json
"reservations": [
  { "hw-address": "b8:27:eb:12:34:56", "ip-address": "192.168.1.51", "hostname": "pi-beamer" }
]
```

**OpenWrt** (`/etc/config/dhcp`):

```text
config host
    option name 'pi-beamer'
    option mac 'b8:27:eb:12:34:56'
    option ip '192.168.1.51'
```

**FRITZ!Box:** *Heimnetz → Netzwerk → Gerät bearbeiten → „Diesem Netzwerkgerät immer die gleiche IPv4-Adresse zuweisen“*.

**Statische IP unter Ubuntu (Netplan)**, nur wenn keine Reservierung möglich ist:

```yaml
# /etc/netplan/01-static.yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses: [192.168.1.20/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [192.168.1.1]
```

```bash
sudo netplan try    # mit automatischem Rollback, falls die Verbindung abreißt
```

---

## Hostname setzen

```bash
sudo hostnamectl set-hostname pi-beamer
# /etc/hosts anpassen: 127.0.1.1  pi-beamer
```

Namenskonvention: klein, mit Bindestrich (`kebab-case`), sprechend und ohne Umlaute: `pi-beamer`, `nas-lab`, `vm-openhab`. Siehe [Namenskonventionen](../Code-Formatierung/Namenskonventionen.md).

---

## Checkliste

- [ ] MAC-Adresse ermittelt (keine randomisierte MAC)
- [ ] IP aus dem Adressplan, außerhalb des DHCP-Pools bzw. als Reservierung
- [ ] Reservierung angelegt und getestet (Gerät neu gestartet)
- [ ] Hostname gesetzt, DNS-Eintrag vorhanden
- [ ] Inventar / Adressliste aktualisiert
- [ ] Gerät in Ansible aufgenommen

---
