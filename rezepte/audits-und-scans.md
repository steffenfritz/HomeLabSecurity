# Audits & Scans

**Level:** 🟠 Medium → 🔴 High · **Zeit:** 1–2 h · **Checkliste:** LOW-ZUG-02, MED-SYS-01, MED-CON-04, HIGH-SYS-01, HIGH-UPD-01, HIGH-IR-04

## Warum?

Was man nicht prüft, weiß man nicht. Diese Werkzeuge zeigen dir dein Homelab aus der Sicht eines Angreifers – und vergleichen es mit bewährten Härtungsempfehlungen.

> Scanne **nur eigene** Systeme. Scans fremder Systeme ohne Erlaubnis können strafbar sein.

## 1. Was ist erreichbar? – nmap

```bash
# Von außen (Hotspot/VPS) auf die eigene öffentliche IP
nmap -Pn -p- <öffentliche-IP>

# Im LAN: Welche Geräte gibt es?
nmap -sn 192.168.10.0/24

# Welche Dienste in welcher Version laufen auf einem Host?
nmap -sV -p- 192.168.10.5
```

Ergebnis mit deiner Dienste-Liste ([Inventar](../checkliste/01-low.md)) abgleichen. Unbekannte Geräte oder Ports? Nachgehen.

## 2. Wie gut ist der Host gehärtet? – Lynis

```bash
sudo apt install lynis      # oder aktuelle Version: git clone https://github.com/CISOfy/lynis
sudo lynis audit system
```

Am Ende stehen **Warnings**, **Suggestions** und ein **Hardening Index**. Vorgehen:

1. Alle *Warnings* anschauen und beheben.
2. *Suggestions* nach Aufwand/Nutzen sortieren – nicht alles ist im Homelab sinnvoll.
3. Erneut laufen lassen, Index vergleichen. Bewusst nicht umgesetzte Punkte in `/etc/lynis/custom.prf` mit `skip-test=…` ausblenden und begründen.

Details: `/var/log/lynis-report.dat`.

## 3. Docker-Konfiguration – docker-bench-security

Prüft Host und Daemon gegen den **CIS Docker Benchmark**. Aufruf siehe [Container härten](container-haerten.md#4-prüfen-mit-werkzeugen).

## 4. Bekannte Schwachstellen in Images – Trivy

```bash
# Einzelnes Image
trivy image --severity HIGH,CRITICAL nginx:1.27.3

# Alle laufenden Images
docker ps --format '{{.Image}}' | sort -u | xargs -n1 trivy image --severity CRITICAL --quiet

# Compose-/Konfigurationsdateien auf Fehlkonfigurationen und Secrets
trivy fs --scanners misconfig,secret .
```

**Einordnen:** Nicht jede CVE ist relevant. Fragen: Ist die betroffene Komponente überhaupt erreichbar? Gibt es ein Update (`Fixed Version`)? Steht sie im **CISA KEV-Katalog** (wird aktiv ausgenutzt)? Dann sofort handeln.

Alternative: **Grype** (+ **Syft** für SBOMs).

## 5. Netzwerkweite Schwachstellenscans – Greenbone / OpenVAS (High)

Greenbone Community Edition scannt alle Hosts im Netz auf bekannte Schwachstellen, veraltete Software und Fehlkonfigurationen. Installation per Docker Compose laut offizieller Doku (*Greenbone Community Containers*).

- Ressourcenhungrig (mehrere GB RAM), Feed-Synchronisation dauert beim ersten Start lange.
- Scans im Monatsrhythmus planen, Ergebnisse mit dem Patch-Tag abarbeiten.
- Alternative ohne eigenen Scanner: Wazuh *Vulnerability Detection* über die Agenten → [Wazuh](wazuh.md).

## 6. Härtung nach Benchmark (High)

- **dev-sec/ansible-collection-hardening** – Ansible-Rollen für OS-, SSH-, nginx-, MySQL-Härtung, angelehnt an CIS/DevSec-Baselines.
- **OpenSCAP** – prüft und härtet gegen SCAP-Profile (CIS, ANSSI, STIG):
  ```bash
  sudo apt install openscap-scanner ssg-debian   # Paketnamen je nach Distribution
  oscap info /usr/share/xml/scap/ssg/content/ssg-debian12-ds.xml
  ```
- **BSI IT-Grundschutz-Kompendium** – deutsche Referenz, z. B. Bausteine *SYS.1.3 Server unter Linux* und *APP.4.4 Kubernetes*/*SYS.1.6 Containerisierung*.

> ⚠️ Benchmarks sind für Unternehmen geschrieben. 100 % zu erreichen ist im Homelab weder nötig noch sinnvoll. Bewusst entscheiden, dokumentieren, weitermachen.

## Prüfen

- [ ] Externer nmap-Scan dokumentiert, keine Überraschungen.
- [ ] Lynis-Index vorher/nachher notiert.
- [ ] Keine CRITICAL-CVEs mit verfügbarem Fix in laufenden Images.

## Fallstricke

- Scanner melden viel. **Priorisieren** statt in der Liste zu ertrinken: exponierte Dienste zuerst, dann KEV, dann CVSS.
- Ein einmaliger Scan ist eine Momentaufnahme. Regelmäßig wiederholen (Kalender, Cron, CI).
