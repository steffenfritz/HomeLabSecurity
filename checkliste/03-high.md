# 🔴 Level High – Erkennen, härten, reagieren

**Voraussetzung:** [🟠 Medium](02-medium.md) ist erledigt.

**Ziel:** Du gehst davon aus, dass irgendwann etwas durchrutscht („Assume Breach“) – und willst es **erkennen**, **eingrenzen** und **sauber wiederherstellen** können. Systeme sind nach anerkannten Benchmarks gehärtet.

**Aufwand:** fortlaufend · **Wartung:** regelmäßig, mehrere Stunden pro Monat

> ⚠️ **KISS-Check:** Dieses Level ist für sensible Daten, öffentlich erreichbare Dienste – oder weil du beruflich lernen willst. Ein halb gewartetes SIEM ist schlechter als ein sauber gepflegtes Level *Medium*. Wazuh, Suricata & Co. brauchen spürbar RAM, CPU und Aufmerksamkeit.

---

## Inventar & Dokumentation

- [ ] **HIGH-INV-01** – Vollständiges Asset-Inventar inkl. Software-Versionen (z. B. NetBox, Wazuh-Inventar, SBOMs).
- [ ] **HIGH-INV-02** – Gesamte Infrastruktur als Code (Ansible, OpenTofu/Terraform, NixOS); ein Host kann jederzeit neu aufgebaut werden.

## Zugang von außen

- [ ] **HIGH-ZUG-01** – **Zero Trust**: Kein Dienst vertraut dem LAN. Jeder Zugriff wird authentifiziert – auch intern.
- [ ] **HIGH-ZUG-02** – Admin-Zugriff nur über einen dedizierten **Jump-Host** oder eine Admin-Workstation.
- [ ] **HIGH-ZUG-03** – Öffentlich erreichbare Dienste in einer **DMZ** ohne Zugriff ins interne Netz.

## Netzwerk

- [ ] **HIGH-NET-01** – **IDS/IPS** (Suricata auf OPNsense, Zeek, Security Onion) mit gepflegten Regeln.
- [ ] **HIGH-NET-02** – **Egress-Filter** auch für Server: ausgehend nur erlaubte Ziele/Ports (erschwert Datenabfluss und C2-Kommunikation).
- [ ] **HIGH-NET-03** – DNS-Anfragen werden geloggt und ausgewertet; DoH/DoT-Umgehung im LAN wird blockiert.
- [ ] **HIGH-NET-04** – Netzwerkfluss-Analyse (ntopng, NetFlow/IPFIX) für ungewöhnlichen Traffic.
- [ ] **HIGH-NET-05** – 802.1X oder zumindest MAC-basierte VLAN-Zuweisung; ungenutzte Switch-Ports deaktiviert.

## Betriebssystem & Hosts

- [ ] **HIGH-SYS-01** – Härtung nach **CIS Benchmark** oder **BSI IT-Grundschutz** (z. B. per `dev-sec/ansible-collection-hardening`, OpenSCAP).
  → [Rezept: Audits & Scans](../rezepte/audits-und-scans.md)
- [ ] **HIGH-SYS-02** – **Festplattenverschlüsselung** auch auf Servern, mit Remote-Unlock (Dropbear im initramfs, Clevis/Tang).
- [ ] **HIGH-SYS-03** – Secure Boot aktiv, wo möglich.
- [ ] **HIGH-SYS-04** – **SSH mit FIDO2-Hardware-Keys** (`ed25519-sk`), Zertifikate statt `authorized_keys` (step-ca, OpenSSH CA).
  → [Rezept: SSH härten](../rezepte/ssh-haerten.md#high-fido2-hardware-keys)
- [ ] **HIGH-SYS-05** – AppArmor/SELinux im Enforcing-Modus.
- [ ] **HIGH-SYS-06** – `auditd` mit sinnvollen Regeln, Logs gehen an das SIEM.

## Container

- [ ] **HIGH-CON-01** – **Rootless** Docker oder Podman.
- [ ] **HIGH-CON-02** – Alle Images per Digest gepinnt; Signaturen (cosign) werden geprüft, wo verfügbar.
- [ ] **HIGH-CON-03** – Runtime-Erkennung (Falco) für verdächtiges Verhalten in Containern.
- [ ] **HIGH-CON-04** – Eigene Images: minimale Basis (distroless, Alpine, Wolfi), SBOM, Scan im CI.

## Konten & Passwörter

- [ ] **HIGH-IAM-01** – **Hardware-Security-Keys** (FIDO2) für IdP, Passwortmanager, E-Mail, Registrar – mit Ersatzschlüssel.
- [ ] **HIGH-IAM-02** – Secrets-Management (OpenBao/Vault, Infisical) mit **Rotation**.
- [ ] **HIGH-IAM-03** – Regelmäßiger Review aller Konten, API-Tokens und SSH-Schlüssel; Unbenutztes wird entfernt.

## Backup

- [ ] **HIGH-BAK-01** – Eine **Offline-Kopie** (Air Gap): z. B. rotierende USB-Platten, eine davon immer außer Haus.
- [ ] **HIGH-BAK-02** – Automatisierte Restore-Tests (Backup wird regelmäßig automatisch in eine Test-VM zurückgespielt und geprüft).
- [ ] **HIGH-BAK-03** – Backup-Infrastruktur hat **eigene, getrennte Zugangsdaten** und ist nicht Mitglied im normalen IdP.

## Updates

- [ ] **HIGH-UPD-01** – **Schwachstellen-Management**: regelmäßige Scans (Greenbone/OpenVAS, Wazuh Vulnerability Detection, Trivy) mit Priorisierung (z. B. CISA KEV).
- [ ] **HIGH-UPD-02** – Kritische Lücken in exponierten Diensten werden innerhalb von 48 Stunden gepatcht oder der Dienst wird vom Netz genommen.

## Monitoring & Erkennung

- [ ] **HIGH-MON-01** – **SIEM/XDR** (Wazuh) mit Agenten auf allen Hosts: File Integrity Monitoring, Konfigurations-Assessment, Schwachstellen-Erkennung.
  → [Rezept: Wazuh einrichten](../rezepte/wazuh.md)
- [ ] **HIGH-MON-02** – Logs sind **manipulationssicher** gespeichert (Remote, append-only), Aufbewahrung mind. 90 Tage.
- [ ] **HIGH-MON-03** – **Canary Tokens / Honeypots** (OpenCanary, canarytokens.org) als Frühwarnsystem.
- [ ] **HIGH-MON-04** – Regelmäßiger Review der Alarme und Regeln; False Positives werden getunt, nicht ignoriert.

## Physisch & Strom

- [ ] **HIGH-PHY-01** – Rack/Schrank abschließbar; Gehäuseöffnung wird erkannt, wo möglich.
- [ ] **HIGH-PHY-02** – Ersatzhardware für kritische Komponenten (Router, Switch) liegt bereit.

## Notfall & Incident Response

- [ ] **HIGH-IR-01** – **Playbooks** für typische Fälle: kompromittierter Host, Ransomware, gestohlene Zugangsdaten, verlorenes Gerät.
  → [Rezept: Notfallplan](../rezepte/notfallplan.md)
- [ ] **HIGH-IR-02** – Möglichkeit, einen Host per Knopfdruck zu **isolieren** (Firewall-Regel, VLAN-Quarantäne, Wazuh Active Response).
- [ ] **HIGH-IR-03** – Jährliche **Übung**: Ein Dienst wird „verloren“ und nach Plan wiederhergestellt.
- [ ] **HIGH-IR-04** – Selbst-Pentest: externe und interne Scans, Prüfung gegen OWASP-Top-10 für eigene Web-Apps.
