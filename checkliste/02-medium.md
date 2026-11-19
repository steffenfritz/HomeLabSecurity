# 🟠 Level Medium – Segmentieren, zentralisieren, sehen

**Voraussetzung:** [🔵 Low](01-low.md) ist erledigt.

**Ziel:** Ein kompromittiertes Gerät kann nicht das ganze Netz übernehmen. Anmeldungen sind zentral und mit 2FA geschützt. Du siehst den Zustand aller Systeme an einer Stelle. Backups überleben auch Ransomware.

**Aufwand:** mehrere Wochenenden · **Wartung:** ca. 1 Stunde pro Monat

> ⚠️ **KISS-Check:** Ab hier kommen neue Komponenten dazu (Firewall, Reverse Proxy, IdP, Monitoring-Server). Jede davon muss gepatcht und verstanden werden. Setze nur um, was dein Setup wirklich braucht.

---

## Inventar & Dokumentation

- [ ] **MED-INV-01** – **Netzplan**: VLANs, Subnetze, Firewall-Regeln, welche Dienste wo laufen (draw.io, Excalidraw, NetBox oder einfach Markdown).
- [ ] **MED-INV-02** – Hosts sind **reproduzierbar** aufgesetzt (Ansible, Skripte, Cloud-Init, NixOS …), nicht nur per Hand.

## Zugang von außen

- [ ] **MED-ZUG-01** – Mesh-VPN-ACLs nach **Least Privilege**: Tags für Server, Gruppen für Personen, nur benötigte Ports.
- [ ] **MED-ZUG-02** – Key-Expiry für Clients aktiv; für Server bewusst deaktiviert und dokumentiert.
- [ ] **MED-ZUG-03** – Muss etwas öffentlich sein: **nur Port 443** über Reverse Proxy (oder Tunnel), mit SSO/2FA und CrowdSec davor. Niemals Datenbanken, SSH, SMB, RDP, Admin-UIs.
  → [Rezept: Reverse Proxy, TLS & SSO](../rezepte/reverse-proxy-tls-sso.md)

## Netzwerk

- [ ] **MED-NET-01** – **VLAN-Segmentierung**: mindestens Management, Server, Clients, IoT, Gäste.
  → [Rezept: Netzwerk segmentieren](../rezepte/netzwerk-segmentierung.md)
- [ ] **MED-NET-02** – Inter-VLAN-Routing **default deny**, nur explizite Freigaben (z. B. Clients → Server: 443).
- [ ] **MED-NET-03** – **Management-Interfaces** (Proxmox, IPMI/iDRAC/iLO, Switch, AP, NAS-Admin) nur aus dem Management-VLAN erreichbar.
- [ ] **MED-NET-04** – **IPMI/BMC niemals** im Internet und nicht im normalen LAN.
- [ ] **MED-NET-05** – IoT-VLAN: kein Zugriff aufs LAN; Internetzugriff nur, wo nötig.
- [ ] **MED-NET-06** – Eigener DNS-Resolver; IoT-Geräte werden gezwungen, ihn zu benutzen (DNS-Redirect auf Port 53).
- [ ] **MED-NET-07** – Interne Dienste per **HTTPS** mit gültigem Zertifikat (Wildcard via DNS-01 oder `tailscale serve`), keine Zertifikatswarnungen, die man „wegklickt“.
  → [Rezept: Reverse Proxy, TLS & SSO](../rezepte/reverse-proxy-tls-sso.md)

## Betriebssystem & Hosts

- [ ] **MED-SYS-01** – **Lynis-Audit** auf jedem Host, wichtigste Empfehlungen umgesetzt.
  → [Rezept: Audits & Scans](../rezepte/audits-und-scans.md)
- [ ] **MED-SYS-02** – Neustarts nach Kernel-Updates geplant (`needrestart`, automatischer Reboot-Zeitpunkt).
- [ ] **MED-SYS-03** – Proxmox/Hypervisor: eingebaute Firewall aktiv, VMs/LXCs getrennt nach Vertrauensstufe; unprivilegierte LXC-Container.
- [ ] **MED-SYS-04** – Logs werden **zentral** gesammelt (Loki, Graylog, syslog-Server) und mindestens 30 Tage aufbewahrt.

## Container

- [ ] **MED-CON-01** – Container **gehärtet**: Non-Root-User, `cap_drop: [ALL]`, `no-new-privileges`, wo möglich `read_only`.
  → [Rezept: Container härten](../rezepte/container-haerten.md)
- [ ] **MED-CON-02** – Eigene Docker-Netzwerke pro Stack; Datenbanken in `internal: true`-Netzen ohne veröffentlichte Ports.
- [ ] **MED-CON-03** – Ressourcen-Limits (Speicher, PIDs) für Container.
- [ ] **MED-CON-04** – **Image-Scanning** mit Trivy oder Grype vor dem Update.
  → [Rezept: Audits & Scans](../rezepte/audits-und-scans.md)
- [ ] **MED-CON-05** – **Renovate** (oder ähnliches) erstellt Update-PRs für die Compose-Dateien im Git-Repo.
  → [Rezept: Docker-Images pinnen](../rezepte/docker-images-pinnen.md#renovate)
- [ ] **MED-CON-06** – Kritische Images zusätzlich per **Digest** (`@sha256:…`) gepinnt.

## Konten & Passwörter

- [ ] **MED-IAM-01** – **Single Sign-On** (Authelia, Authentik, Pocket ID, Keycloak) mit 2FA vor Diensten ohne eigene gute Authentifizierung.
  → [Rezept: Reverse Proxy, TLS & SSO](../rezepte/reverse-proxy-tls-sso.md)
- [ ] **MED-IAM-02** – **Passkeys/WebAuthn** statt TOTP, wo möglich.
- [ ] **MED-IAM-03** – Secrets in Git nur verschlüsselt (SOPS + age) oder gar nicht.
  → [Rezept: Passwörter, 2FA & Secrets](../rezepte/passwoerter-2fa-secrets.md)
- [ ] **MED-IAM-04** – Getrennte Admin- und Alltagskonten (z. B. am NAS, im IdP).

## Backup

- [ ] **MED-BAK-01** – **Unveränderliche Backups** (append-only, S3 Object Lock, Proxmox Backup Server mit eigenen Rechten) – Ransomware darf Backups nicht löschen können.
  → [Rezept: Backups nach 3-2-1](../rezepte/backup-3-2-1.md#ransomware-sicher-append-only)
- [ ] **MED-BAK-02** – **Pull statt Push** oder getrennte Zugangsdaten: Ein kompromittierter Server kann sein Backup-Ziel nicht manipulieren.
- [ ] **MED-BAK-03** – **Vierteljährlicher Restore-Test** eines kompletten Dienstes, dokumentiert.
- [ ] **MED-BAK-04** – VMs/LXCs werden zusätzlich als Ganzes gesichert (Proxmox Backup Server).

## Updates

- [ ] **MED-UPD-01** – Übersicht über ausstehende Updates aller Hosts im Monitoring.
- [ ] **MED-UPD-02** – Abonnierte Security-Advisories für die wichtigsten eingesetzten Produkte (GitHub „Watch → Releases“, Mailinglisten, RSS).

## Monitoring

- [ ] **MED-MON-01** – **Zentrales Monitoring** (Checkmk, Zabbix, Prometheus + Grafana, Beszel) für alle Hosts: CPU, RAM, Platten, Dienste, Temperaturen, USV.
  → [Rezept: Monitoring](../rezepte/monitoring.md)
- [ ] **MED-MON-02** – Alarme sind **sinnvoll eingestellt** – wenige, aber wichtige. Alarmmüdigkeit ist ein Sicherheitsrisiko.
- [ ] **MED-MON-03** – Fehlgeschlagene Logins (SSH, IdP, Reverse Proxy) werden ausgewertet und alarmiert.
- [ ] **MED-MON-04** – Monitoring überwacht sich nicht nur selbst: Ein externer Dead-Man's-Switch (Healthchecks) meldet, wenn das Monitoring ausfällt.

## Physisch & Strom

- [ ] **MED-PHY-01** – Überspannungsschutz; USV-Akku regelmäßig getestet.
- [ ] **MED-PHY-02** – BIOS/UEFI-Passwort auf Servern; Boot von USB nur mit Passwort.

## Notfall

- [ ] **MED-IR-01** – Schriftlicher Notfallplan inkl. **Wiederaufbau-Reihenfolge** (Netzwerk → DNS → IdP → Passwortmanager → Rest).
  → [Rezept: Notfallplan](../rezepte/notfallplan.md)
- [ ] **MED-IR-02** – Eine Vertrauensperson weiß, wo der Notfallzettel liegt (Stichwort: digitaler Nachlass).

---

Weiter mit [🔴 High](03-high.md) – aber nur, wenn du es wirklich brauchst oder lernen willst.
