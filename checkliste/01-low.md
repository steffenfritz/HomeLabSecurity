# 🔵 Level Low – Grundhygiene

**Voraussetzung:** [🟢 KISS](00-kiss.md) ist erledigt.

**Ziel:** Jeder Host ist für sich vernünftig abgesichert, Backups laufen zuverlässig und du merkst, wenn etwas schiefgeht.

**Aufwand:** ein Wochenende · **Wartung:** wenige Minuten pro Monat

---

## Inventar & Dokumentation

- [ ] **LOW-INV-01** – Liste der Dienste mit: Zweck, Host, Ports, wer benutzt ihn, wo liegen die Daten, wie wird er gesichert.
- [ ] **LOW-INV-02** – **Notfallzettel**: Wie komme ich an meine Systeme, wenn VPN, DNS oder der Passwortmanager-Server ausfallen? (Lokaler Zugriff, Recovery-Codes, Router-Zugang.)
  → [Rezept: Notfallplan](../rezepte/notfallplan.md)
- [ ] **LOW-INV-03** – `docker-compose.yml` und Konfigurationen liegen in einem **Git-Repository** (ohne Secrets!).

## Zugang von außen

- [ ] **LOW-ZUG-01** – Mesh-VPN mit **Zugriffsregeln (ACLs)**: Nicht jedes Gerät im Tailnet darf alles erreichen.
- [ ] **LOW-ZUG-02** – Externer **Portscan** deiner öffentlichen IPv4 *und* IPv6 zeigt keine offenen Ports (oder nur bewusst offene).
  → [Rezept: Ports schließen & Mesh-VPN](../rezepte/ports-schliessen-mesh-vpn.md)
- [ ] **LOW-ZUG-03** – Bei IPv6: Router-Firewall lässt **keine eingehenden Verbindungen** zu Hosts im LAN zu (Freigaben pro Gerät prüfen).

## Netzwerk

- [ ] **LOW-NET-01** – **Host-Firewall** auf jedem Server: eingehend alles verbieten, nur benötigte Ports erlauben.
  → [Rezept: Host-Firewall](../rezepte/host-firewall.md)
- [ ] **LOW-NET-02** – **Docker-Ports nur an `127.0.0.1` oder die VPN-IP binden** – Docker umgeht `ufw`!
  → [Rezept: Host-Firewall](../rezepte/host-firewall.md#docker-umgeht-ufw)
- [ ] **LOW-NET-03** – Feste IPs (DHCP-Reservierungen) für Server und Infrastruktur.
- [ ] **LOW-NET-04** – Optional: DNS-Filter (Pi-hole, AdGuard Home) für Werbung, Tracking und bekannte Malware-Domains.

## Betriebssystem & Hosts

- [ ] **LOW-SYS-01** – Kein Arbeiten als `root`: persönlicher Benutzer mit `sudo`.
- [ ] **LOW-SYS-02** – SSH: kein Root-Login, `AllowUsers` gesetzt, nur Schlüssel.
  → [Rezept: SSH härten](../rezepte/ssh-haerten.md)
- [ ] **LOW-SYS-03** – Nicht benötigte Pakete und Dienste entfernt (`systemctl list-units --type=service --state=running`).
- [ ] **LOW-SYS-04** – Zeitsynchronisation aktiv (wichtig für Logs, TLS und 2FA).
- [ ] **LOW-SYS-05** – Ist doch etwas öffentlich erreichbar: **Fail2ban oder CrowdSec** davor.
  → [Rezept: Fail2ban & CrowdSec](../rezepte/fail2ban-crowdsec.md)
- [ ] **LOW-SYS-06** – Geräte, die das Haus verlassen (Laptop, Handy), sind **vollverschlüsselt**.
- [ ] **LOW-SYS-07** – Admin-Weboberflächen (Proxmox, NAS, Router, Portainer …) sind **nie** direkt aus dem Internet erreichbar.

## Container

- [ ] **LOW-CON-01** – **Kein `docker.sock`** in Container mounten – oder nur read-only über einen Socket-Proxy.
  → [Rezept: Container härten](../rezepte/container-haerten.md)
- [ ] **LOW-CON-02** – Kein `privileged: true` und kein `network_mode: host` ohne guten Grund.
- [ ] **LOW-CON-03** – Images aus vertrauenswürdigen Quellen: offizielle Images, Projekt-Registry (ghcr.io des Projekts), verifizierte Publisher.
- [ ] **LOW-CON-04** – `.env`-Dateien mit Rechten `600`, nicht im Git.
- [ ] **LOW-CON-05** – **Benachrichtigung bei neuen Image-Versionen** (Diun, What's Up Docker) statt automatischem Update. Watchtower wird nicht mehr gepflegt → durch Diun ersetzen.
  → [Rezept: Docker-Images pinnen](../rezepte/docker-images-pinnen.md)
- [ ] **LOW-CON-06** – Bewusst: Wer in der Gruppe `docker` ist, ist faktisch `root`.

## Konten & Passwörter

- [ ] **LOW-IAM-01** – 2FA für **alle** Dienste, die es anbieten.
- [ ] **LOW-IAM-02** – Recovery-Codes ausgedruckt und sicher verwahrt.
- [ ] **LOW-IAM-03** – Getrennte Konten für Familienmitglieder, keine geteilten Admin-Zugänge.
- [ ] **LOW-IAM-04** – E-Mail-Adressen bei [Have I Been Pwned](https://haveibeenpwned.com) überwachen lassen.

## Backup

- [ ] **LOW-BAK-01** – **3-2-1**: 3 Kopien, 2 verschiedene Medien, 1 außer Haus.
  → [Rezept: Backups nach 3-2-1](../rezepte/backup-3-2-1.md)
- [ ] **LOW-BAK-02** – Backups sind **verschlüsselt** (restic, Borg, Kopia).
- [ ] **LOW-BAK-03** – Datenbanken werden per **Dump** gesichert, nicht als laufende Dateien.
- [ ] **LOW-BAK-04** – **Backup-Monitoring**: Du bekommst eine Meldung, wenn ein Backup *nicht* lief (z. B. Healthchecks).
- [ ] **LOW-BAK-05** – Konfiguration von Router, Switch, NAS exportiert und gesichert.

## Updates

- [ ] **LOW-UPD-01** – **Fester Patch-Termin** (z. B. erster Sonntag im Monat): Container, Proxmox, NAS, Firmware.
  → [Rezept: Automatische Updates](../rezepte/automatische-updates.md)
- [ ] **LOW-UPD-02** – Vor Major-Updates: Changelog lesen, Backup/Snapshot machen.
- [ ] **LOW-UPD-03** – Geräte ohne Sicherheitsupdates (End of Life) identifiziert und ersetzt oder isoliert.

## Monitoring

- [ ] **LOW-MON-01** – Warnungen für: **Plattenplatz, SMART-Werte, RAID/ZFS-Status**, Zertifikatsablauf.
  → [Rezept: Monitoring](../rezepte/monitoring.md)
- [ ] **LOW-MON-02** – Benachrichtigungen landen auf dem Handy (ntfy, Gotify, Telegram, E-Mail) – und werden gelesen.

## Physisch & Strom

- [ ] **LOW-PHY-01** – **USV** für NAS und Server, inkl. automatischem Herunterfahren (NUT).
- [ ] **LOW-PHY-02** – Server/NAS nicht frei zugänglich (z. B. nicht im Flur oder im Gäste-/Kinderzimmer ohne Schutz).

## Notfall

- [ ] **LOW-IR-01** – Du weißt, was du tust, wenn ein System kompromittiert ist (Netzwerk trennen, nicht hektisch löschen, von sauberem Gerät Passwörter ändern).
  → [Rezept: Notfallplan](../rezepte/notfallplan.md)

---

Weiter mit [🟠 Medium](02-medium.md).
