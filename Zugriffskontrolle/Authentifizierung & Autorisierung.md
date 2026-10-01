# 🪪 Authentifizierung & Autorisierung

Zwei Begriffe, die ständig verwechselt werden – auch weil sie auf Englisch beide mit *Auth* abgekürzt werden (*AuthN* und *AuthZ*).

<!-- TOC -->
## Inhaltsverzeichnis

- [Der Unterschied in einem Satz](#der-unterschied-in-einem-satz)
- [Analogie: Hotel](#analogie-hotel)
- [AAA: Authentication, Authorization, Accounting](#aaa-authentication-authorization-accounting)
- [Authentifizierungsfaktoren](#authentifizierungsfaktoren)
- [Beispiele aus verschiedenen Domänen](#beispiele-aus-verschiedenen-domänen)
- [Typische Verfahren und Begriffe](#typische-verfahren-und-begriffe)
- [Häufige Fehler](#häufige-fehler)
<!-- /TOC -->

## Der Unterschied in einem Satz

| | Frage | Beispiel |
| --- | --- | --- |
| **Authentifizierung** (*Authentication*, AuthN) | **Wer bist du?** – Nachweis der Identität | Login mit Passwort, SSH-Schlüssel, Fingerabdruck |
| **Autorisierung** (*Authorization*, AuthZ) | **Was darfst du?** – Prüfung der Berechtigung | Darf dieser Benutzer das Topic `lab/#` lesen? Darf er die Datei löschen? |

Die Reihenfolge ist immer: **erst Authentifizierung, dann Autorisierung.** Ohne zu wissen, wer jemand ist, kann man nicht entscheiden, was er darf.

> 💡 Im Deutschen gibt es zusätzlich die Unterscheidung **Authentisierung** (der Benutzer *weist* seine Identität *nach*, z. B. gibt das Passwort ein) und **Authentifizierung** (das System *prüft* diesen Nachweis). Im Englischen ist beides *authentication*.

---

## Analogie: Hotel

* An der **Rezeption** zeigst du deinen Ausweis → **Authentifizierung**.
* Du bekommst eine **Schlüsselkarte**, die nur dein Zimmer, den Fitnessraum und das Frühstücksrestaurant öffnet → **Autorisierung**.
* Das Schloss protokolliert jede Öffnung → **Accounting**.
* Die Schlüsselkarte ist ein **Token**: Sie beweist, dass du bereits authentifiziert wurdest, ohne dass du jedes Mal den Ausweis zeigen musst.

---

## AAA: Authentication, Authorization, Accounting

Das **AAA-Modell** ergänzt die beiden Begriffe um das **Accounting** (Protokollierung: Wer hat wann was getan?). Es stammt aus der Netzwerkwelt (RADIUS, TACACS+, z. B. bei WLAN mit WPA2-Enterprise), gilt aber allgemein.

---

## Authentifizierungsfaktoren

| Faktor | Beispiel |
| --- | --- |
| **Wissen** (*something you know*) | Passwort, PIN, Sicherheitsfrage |
| **Besitz** (*something you have*) | Smartphone (TOTP-App), Hardware-Token (YubiKey), Chipkarte, SSH-Private-Key |
| **Inhärenz** (*something you are*) | Fingerabdruck, Gesichtserkennung |

**MFA / 2FA** (Multi-Faktor-Authentifizierung) kombiniert Faktoren **verschiedener** Kategorien. Zwei Passwörter sind keine 2FA.

---

## Beispiele aus verschiedenen Domänen

| Domäne | Authentifizierung | Autorisierung |
| --- | --- | --- |
| **Forum** | Login mit E-Mail + Passwort | Rollen: Gast darf lesen, Mitglied darf schreiben, Moderator darf löschen, Admin darf alles |
| **Linux** | Login mit Passwort oder SSH-Schlüssel (PAM) | Dateirechte `rwx`, Gruppen, `sudoers`, POSIX-ACL |
| **Datenbank** | `CREATE USER ... IDENTIFIED BY ...` | `GRANT SELECT ON db.table TO user` |
| **MQTT (Mosquitto)** | `password_file`, Client-Zertifikat | `acl_file` (Topic-Rechte) |
| **openHAB** | Benutzerkonto, API-Token | Rolle `administrator` vs. `user` |
| **Web-API** | OAuth 2.0 / OpenID Connect, API-Key | Scopes im Token (`read:devices`), Rollen |
| **WLAN** | WPA2/WPA3-Passwort oder 802.1X (Benutzer + Zertifikat) | VLAN-Zuordnung, Gastnetz |
| **Git-Hosting** | SSH-Key, Personal Access Token | Repo-Rollen: Read, Write, Maintain, Admin |
| **Firewall** | (meist keine – sie kennt nur IPs/Ports) | Regeln: Quelle, Ziel, Port → erlauben/verbieten |

Die Firewall zeigt: Manche Systeme **autorisieren ohne zu authentifizieren**. Eine IP-Adresse ist keine Identität, sie lässt sich fälschen oder teilen. Deshalb ersetzt eine Firewall-Regel nie einen Login.

---

## Typische Verfahren und Begriffe

| Begriff | Bedeutung |
| --- | --- |
| **Session / Cookie** | Nach dem Login merkt sich der Server die Authentifizierung über eine Sitzungs-ID im Cookie. |
| **Token** | Ausweis nach erfolgreichem Login, z. B. **JWT** (*JSON Web Token*), der Identität und ggf. Rechte enthält und signiert ist. |
| **API-Key** | Langer geheimer Schlüssel für Programme. Authentifiziert meist eine Anwendung, nicht eine Person. |
| **OAuth 2.0** | Standard für **Autorisierung**: Eine Anwendung erhält begrenzten Zugriff im Namen eines Benutzers („Diese App möchte deine Kontakte lesen“). |
| **OpenID Connect (OIDC)** | Aufsatz auf OAuth 2.0 für **Authentifizierung** („Login mit Google/GitHub“). |
| **SSO** | *Single Sign-On*: Einmal anmelden, bei vielen Diensten eingeloggt (z. B. Hochschul-Login über Shibboleth). |
| **LDAP / Active Directory** | Zentrale Benutzerverzeichnisse, gegen die viele Systeme authentifizieren. |
| **PAM** | *Pluggable Authentication Modules*: Authentifizierungs-Framework unter Linux. |

---

## Häufige Fehler

* **Nur authentifizieren, nicht autorisieren:** Jeder eingeloggte Benutzer darf alles (MQTT ohne ACL, Datenbank-Benutzer mit `ALL PRIVILEGES`).
* **Autorisierung nur im Frontend:** Der Button ist ausgeblendet, aber die API nimmt den Request trotzdem an. Autorisierung gehört **auf den Server**.
* **Ein gemeinsamer Account für alle:** Keine Zuordnung im Log möglich, Zugang nicht einzeln entziehbar.
* **Fehlermeldungen verraten zu viel:** „Benutzer existiert nicht“ vs. „Passwort falsch“ hilft Angreifern. Besser: „Benutzername oder Passwort falsch“.

---
