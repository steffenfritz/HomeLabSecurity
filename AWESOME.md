# Awesome Homelab Security [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> Eine kuratierte Liste von Open-Source- und freien Werkzeugen, um ein Homelab abzusichern, zu überwachen und wiederherzustellen.

**Legende** – ab welchem Level ein Tool typischerweise sinnvoll ist (siehe [README](README.md#die-level)):
🟢 KISS · 🔵 Low · 🟠 Medium · 🔴 High · 💶 kommerziell mit kostenloser Stufe · 🏠 selbst gehostet

## Inhalt

- [Fernzugriff & Mesh-VPN](#fernzugriff--mesh-vpn)
- [Tunnel & Exposition](#tunnel--exposition)
- [Reverse Proxy & TLS](#reverse-proxy--tls)
- [Identität, SSO & 2FA](#identität-sso--2fa)
- [Passwörter & Secrets](#passwörter--secrets)
- [Router & Firewall](#router--firewall)
- [DNS](#dns)
- [Monitoring](#monitoring)
- [Benachrichtigungen](#benachrichtigungen)
- [Logging](#logging)
- [SIEM, XDR & Host-Intrusion-Detection](#siem-xdr--host-intrusion-detection)
- [Netzwerk-IDS/IPS & Traffic-Analyse](#netzwerk-idsips--traffic-analyse)
- [Brute-Force-Schutz](#brute-force-schutz)
- [Schwachstellen- & Härtungs-Scanner](#schwachstellen---härtungs-scanner)
- [Container-Sicherheit](#container-sicherheit)
- [Image-Updates](#image-updates)
- [Backup](#backup)
- [Konfigurationsmanagement & Härtung als Code](#konfigurationsmanagement--härtung-als-code)
- [Inventar & Dokumentation](#inventar--dokumentation)
- [Täuschung & Frühwarnung](#täuschung--frühwarnung)
- [Strom & Hardware](#strom--hardware)
- [Wissen & Referenzen](#wissen--referenzen)
- [Andere Awesome-Listen](#andere-awesome-listen)

---

## Fernzugriff & Mesh-VPN

- [Tailscale](https://tailscale.com) – WireGuard-Mesh ohne Portfreigaben, ACLs, MagicDNS, HTTPS-Zertifikate. 🟢 💶
- [NetBird](https://github.com/netbirdio/netbird) – WireGuard-Mesh, komplett Open Source inkl. Control-Plane, SSO-Anbindung. 🟢 💶 🏠
- [Headscale](https://github.com/juanfont/headscale) – Selbst gehostete Tailscale-Control-Plane. 🟠 🏠
- [WireGuard](https://www.wireguard.com) – Schnelles, schlankes VPN-Protokoll; Basis der meisten Mesh-VPNs. 🔵
- [wg-easy](https://github.com/wg-easy/wg-easy) – WireGuard-Server mit Weboberfläche im Docker-Container. 🔵 🏠
- [ZeroTier](https://www.zerotier.com) – Virtuelles Layer-2-Netz über das Internet. 🟢 💶
- [Nebula](https://github.com/slackhq/nebula) – Overlay-Netz mit Zertifikaten, von Slack entwickelt. 🟠 🏠

## Tunnel & Exposition

Wenn etwas öffentlich erreichbar sein muss – ohne Port zu Hause zu öffnen.

- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) – Ausgehender Tunnel zu Cloudflare; TLS wird bei Cloudflare terminiert. 🔵 💶
- [Tailscale Funnel](https://tailscale.com/kb/1223/funnel) – Einzelne Dienste aus dem Tailnet öffentlich machen. 🔵 💶
- [Pangolin](https://github.com/fosrl/pangolin) – Selbst gehosteter Tunnel-Reverse-Proxy mit Auth, als Cloudflare-Tunnel-Alternative auf einem VPS. 🟠 🏠

## Reverse Proxy & TLS

- [Caddy](https://caddyserver.com) – Reverse Proxy mit automatischem HTTPS, sehr kurze Konfiguration. 🔵 🏠
- [Traefik](https://github.com/traefik/traefik) – Reverse Proxy mit Docker-Label-Konfiguration. 🟠 🏠
- [Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager) – Nginx mit Weboberfläche und Let's Encrypt. 🟢 🏠
- [step-ca](https://github.com/smallstep/certificates) – Eigene private CA mit ACME, auch für SSH-Zertifikate. 🔴 🏠
- [Certbot](https://github.com/certbot/certbot) – Klassischer ACME-Client für Let's Encrypt. 🔵

## Identität, SSO & 2FA

- [Authelia](https://github.com/authelia/authelia) – Forward-Auth-Portal mit 2FA (TOTP, WebAuthn) und OIDC. 🟠 🏠
- [Authentik](https://github.com/goauthentik/authentik) – Identity Provider mit OIDC, SAML, LDAP, Proxy-Outpost. 🟠 🏠
- [Pocket ID](https://github.com/pocket-id/pocket-id) – Schlanker OIDC-Provider ausschließlich mit Passkeys. 🟠 🏠
- [Keycloak](https://github.com/keycloak/keycloak) – Enterprise-IAM, umfangreich. 🔴 🏠
- [LLDAP](https://github.com/lldap/lldap) – Leichtgewichtiger LDAP-Server für Benutzerverwaltung. 🟠 🏠
- [Kanidm](https://github.com/kanidm/kanidm) – Moderner Identity-Server mit OIDC, LDAP, Passkeys, in Rust. 🔴 🏠

## Passwörter & Secrets

- [Bitwarden](https://bitwarden.com) – Open-Source-Passwortmanager, auch für Familien. 🟢 💶
- [Vaultwarden](https://github.com/dani-garcia/vaultwarden) – Ressourcenschonender, Bitwarden-kompatibler Server. 🔵 🏠
- [KeePassXC](https://keepassxc.org) – Lokaler, dateibasierter Passwortmanager. 🟢
- [SOPS](https://github.com/getsops/sops) – Secrets in YAML/JSON/ENV verschlüsselt im Git. 🟠
- [age](https://github.com/FiloSottile/age) – Einfache, moderne Dateiverschlüsselung, oft mit SOPS genutzt. 🟠
- [Infisical](https://github.com/Infisical/infisical) – Secrets-Management-Plattform. 🔴 🏠 💶
- [OpenBao](https://github.com/openbao/openbao) – Community-Fork von HashiCorp Vault für Secrets und PKI. 🔴 🏠
- [Gitleaks](https://github.com/gitleaks/gitleaks) – Findet versehentlich committete Secrets in Git-Repos. 🔵

## Router & Firewall

- [OPNsense](https://opnsense.org) – Open-Source-Firewall/Router mit VLANs, Plugins (CrowdSec, Suricata, WireGuard). 🟠 🏠
- [pfSense CE](https://www.pfsense.org) – Bewährte Firewall-Distribution. 🟠 🏠
- [OpenWrt](https://openwrt.org) – Linux für Router und Access Points, VLAN-fähig. 🟠 🏠
- [IPFire](https://www.ipfire.org) – Firewall-Distribution mit Zonen-Konzept. 🟠 🏠
- [ufw](https://launchpad.net/ufw) – Einfache Host-Firewall für Debian/Ubuntu. 🔵
- [firewalld](https://firewalld.org) – Zonenbasierte Host-Firewall (Fedora/RHEL). 🔵

## DNS

- [Pi-hole](https://github.com/pi-hole/pi-hole) – DNS-Filter für Werbung, Tracking, Malware. 🔵 🏠
- [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) – DNS-Filter mit DoH/DoT, Client-Regeln. 🔵 🏠
- [Technitium DNS Server](https://github.com/TechnitiumSoftware/DnsServer) – Vollwertiger DNS-Server mit Blocklisten, Zonen, DoH. 🟠 🏠
- [Unbound](https://github.com/NLnetLabs/unbound) – Validierender, rekursiver Resolver (DNSSEC). 🟠 🏠
- [Blocky](https://github.com/0xERR0R/blocky) – Leichtgewichtiger DNS-Proxy/-Filter, Konfiguration als YAML. 🔵 🏠

## Monitoring

- [Uptime Kuma](https://github.com/louislam/uptime-kuma) – Uptime-Checks mit schöner UI und vielen Benachrichtigungskanälen. 🟢 🏠
- [Gatus](https://github.com/TwiN/gatus) – Uptime-/Health-Checks als Konfigurationsdatei, Statusseite. 🔵 🏠
- [Healthchecks](https://github.com/healthchecks/healthchecks) – Dead-Man's-Switch für Cronjobs und Backups. 🟢 🏠 💶
- [Beszel](https://github.com/henrygd/beszel) – Leichtgewichtiges Host- und Docker-Monitoring. 🔵 🏠
- [Netdata](https://github.com/netdata/netdata) – Echtzeit-Metriken pro Sekunde, Null-Konfiguration. 🔵 🏠 💶
- [Checkmk](https://checkmk.com) – Umfassendes IT-Monitoring mit Autodiscovery, SNMP, kostenlose Raw-/Community-Edition. 🟠 🏠
- [Zabbix](https://www.zabbix.com) – Enterprise-Monitoring mit Templates für fast alles. 🟠 🏠
- [Icinga](https://icinga.com) – Nagios-kompatibles Monitoring, stark in DACH verbreitet. 🟠 🏠
- [LibreNMS](https://github.com/librenms/librenms) – Netzwerk-Monitoring per SNMP (Switches, Router, APs). 🟠 🏠
- [Prometheus](https://github.com/prometheus/prometheus) – Metrik-Datenbank und Alerting, Cloud-Native-Standard. 🟠 🏠
- [Grafana](https://github.com/grafana/grafana) – Dashboards für Prometheus, Loki und viele mehr. 🟠 🏠
- [node_exporter](https://github.com/prometheus/node_exporter) – Host-Metriken für Prometheus. 🟠 🏠
- [Scrutiny](https://github.com/AnalogJ/scrutiny) – SMART-Monitoring mit Weboberfläche und Ausfall-Prognose. 🔵 🏠
- [smartmontools](https://www.smartmontools.org) – `smartctl`/`smartd` für Plattenzustand. 🟢

## Benachrichtigungen

- [ntfy](https://github.com/binwiederhier/ntfy) – Push-Benachrichtigungen per HTTP, App für Android/iOS. 🟢 🏠 💶
- [Gotify](https://github.com/gotify/server) – Selbst gehosteter Push-Server. 🔵 🏠
- [Apprise](https://github.com/caronc/apprise) – Eine API für über 100 Benachrichtigungsdienste. 🔵 🏠

## Logging

- [Dozzle](https://github.com/amir20/dozzle) – Echtzeit-Logviewer für Docker-Container. 🟢 🏠
- [Grafana Loki](https://github.com/grafana/loki) – Log-Aggregation, ressourcenschonend, mit Grafana. 🟠 🏠
- [Grafana Alloy](https://github.com/grafana/alloy) – Collector für Logs/Metriken (Nachfolger von Promtail/Grafana Agent). 🟠 🏠
- [Graylog](https://github.com/Graylog2/graylog2-server) – Zentrales Log-Management mit Suche und Alerts. 🟠 🏠
- [Vector](https://github.com/vectordotdev/vector) – Schnelle Pipeline zum Sammeln und Weiterleiten von Logs. 🟠 🏠
- [OpenSearch](https://github.com/opensearch-project/OpenSearch) – Such- und Analyse-Engine (Elasticsearch-Fork). 🔴 🏠

## SIEM, XDR & Host-Intrusion-Detection

- [Wazuh](https://github.com/wazuh/wazuh) – SIEM/XDR: FIM, CIS-Assessment, Schwachstellenerkennung, Active Response. → [Rezept](rezepte/wazuh.md) 🔴 🏠
- [Security Onion](https://github.com/Security-Onion-Solutions/securityonion) – Komplette Distribution für Threat Hunting und NSM. 🔴 🏠
- [OSSEC](https://github.com/ossec/ossec-hids) – Klassisches HIDS, Ursprung von Wazuh. 🟠 🏠
- [Falco](https://github.com/falcosecurity/falco) – Runtime-Erkennung für Container und Kubernetes (eBPF). 🔴 🏠
- [osquery](https://github.com/osquery/osquery) – Betriebssystem per SQL abfragen. 🔴
- [AIDE](https://github.com/aide/aide) – Schlankes File Integrity Monitoring. 🟠
- [rkhunter](https://rkhunter.sourceforge.net) – Rootkit-Scanner. 🔵

## Netzwerk-IDS/IPS & Traffic-Analyse

- [Suricata](https://github.com/OISF/suricata) – IDS/IPS, als Plugin in OPNsense/pfSense verfügbar. 🔴 🏠
- [Zeek](https://github.com/zeek/zeek) – Netzwerk-Analyse-Framework, erzeugt reichhaltige Verbindungslogs. 🔴 🏠
- [Snort](https://www.snort.org) – Klassisches IDS/IPS. 🔴 🏠
- [ntopng](https://github.com/ntop/ntopng) – Traffic-Analyse und Flow-Monitoring mit Weboberfläche. 🟠 🏠 💶
- [Arkime](https://github.com/arkime/arkime) – Vollständige Paketaufzeichnung und -suche. 🔴 🏠

## Brute-Force-Schutz

- [CrowdSec](https://github.com/crowdsecurity/crowdsec) – Log-basierte Erkennung mit Community-Blocklisten und Bouncern für Proxies/Firewalls. → [Rezept](rezepte/fail2ban-crowdsec.md) 🔵 🏠
- [Fail2ban](https://github.com/fail2ban/fail2ban) – Sperrt IPs nach fehlgeschlagenen Logins. 🔵 🏠

## Schwachstellen- & Härtungs-Scanner

- [Nmap](https://nmap.org) – Port- und Dienst-Scanner, das Standardwerkzeug. 🔵
- [Lynis](https://github.com/CISOfy/lynis) – Sicherheitsaudit für Linux/Unix mit konkreten Härtungsvorschlägen. 🟠
- [Greenbone Community Edition / OpenVAS](https://github.com/greenbone) – Netzwerkweiter Schwachstellenscanner. 🔴 🏠
- [OpenSCAP](https://github.com/OpenSCAP/openscap) – Compliance-Prüfung gegen CIS/STIG/ANSSI-Profile. 🔴
- [Nuclei](https://github.com/projectdiscovery/nuclei) – Template-basierter Scanner für Web-Schwachstellen. 🔴
- [testssl.sh](https://github.com/testssl/testssl.sh) – TLS-Konfiguration von Diensten prüfen. 🟠
- [ssh-audit](https://github.com/jtesta/ssh-audit) – SSH-Server-Konfiguration prüfen. 🔵
- [Shodan](https://www.shodan.io) – Was sieht das Internet von deiner IP? 🟢 💶

## Container-Sicherheit

- [Trivy](https://github.com/aquasecurity/trivy) – Scanner für Images, Dateisysteme, IaC, Secrets, SBOMs. 🟠
- [Grype](https://github.com/anchore/grype) – Schwachstellenscanner für Images und SBOMs. 🟠
- [Syft](https://github.com/anchore/syft) – SBOM-Generator für Images und Dateisysteme. 🔴
- [Docker Bench for Security](https://github.com/docker/docker-bench-security) – Prüfung gegen den CIS Docker Benchmark. 🟠
- [Dockle](https://github.com/goodwithtech/dockle) – Linter für Container-Images nach Best Practices. 🟠
- [docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy) – Beschränkt den Zugriff auf die Docker-API. 🔵 🏠
- [Cosign](https://github.com/sigstore/cosign) – Signaturen von Container-Images prüfen und erstellen. 🔴
- [Podman](https://github.com/containers/podman) – Daemonlose, standardmäßig rootless Container-Engine. 🔴

## Image-Updates

Benachrichtigen statt blind aktualisieren → [Rezept](rezepte/docker-images-pinnen.md).

- [Diun](https://github.com/crazy-max/diun) – Benachrichtigt bei neuen Image-Versionen, aktualisiert nichts selbst. 🔵 🏠
- [What's Up Docker (WUD)](https://github.com/getwud/wud) – Erkennt Updates mit Weboberfläche, optional mit Triggern. 🔵 🏠
- [Renovate](https://github.com/renovatebot/renovate) – Automatische Update-Pull-Requests für Compose-Dateien und mehr. 🟠 🏠
> ⚠️ **Nicht mehr empfohlen:** [Watchtower](https://github.com/containrrr/watchtower) wird nicht mehr gepflegt. Automatische Updates widersprechen außerdem dem Pinnen. Bestehende Installationen am besten durch **Diun** ersetzen.

## Backup

- [restic](https://github.com/restic/restic) – Schnell, verschlüsselt, dedupliziert, viele Backends. → [Rezept](rezepte/backup-3-2-1.md) 🟢
- [rest-server](https://github.com/restic/rest-server) – HTTP-Backend für restic, mit `--append-only`. 🟠 🏠
- [BorgBackup](https://github.com/borgbackup/borg) – Deduplizierendes, verschlüsseltes Backup. 🔵
- [borgmatic](https://github.com/borgmatic-collective/borgmatic) – Konfigurationsdatei-Wrapper für Borg inkl. DB-Dumps und Healthchecks. 🔵
- [Kopia](https://github.com/kopia/kopia) – Verschlüsseltes Backup mit GUI und CLI. 🟢
- [Backrest](https://github.com/garethgeorge/backrest) – Weboberfläche und Scheduler für restic. 🟢 🏠
- [Proxmox Backup Server](https://www.proxmox.com/en/products/proxmox-backup-server/overview) – Inkrementelle, deduplizierte VM/CT-Backups für Proxmox. 🟠 🏠
- [rclone](https://github.com/rclone/rclone) – „rsync für Cloud-Speicher“, auch als restic-Backend. 🔵
- [Duplicati](https://github.com/duplicati/duplicati) – Backup mit Weboberfläche zu vielen Cloud-Zielen. 🟢 🏠

## Konfigurationsmanagement & Härtung als Code

- [Ansible](https://github.com/ansible/ansible) – Agentenlose Automatisierung per SSH. 🟠
- [dev-sec Hardening Collection](https://github.com/dev-sec/ansible-collection-hardening) – Ansible-Rollen für OS-, SSH-, nginx-, MySQL-Härtung. 🔴
- [OpenTofu](https://github.com/opentofu/opentofu) – Infrastructure as Code (Terraform-Fork), auch mit Proxmox-Provider. 🔴
- [NixOS](https://nixos.org) – Deklaratives, reproduzierbares Linux. 🔴
- [Cloud-init](https://github.com/canonical/cloud-init) – Erstkonfiguration von VMs (auch in Proxmox). 🟠

## Inventar & Dokumentation

- [NetBox](https://github.com/netbox-community/netbox) – IPAM und Infrastruktur-Dokumentation. 🟠 🏠
- [Homebox](https://github.com/sysadminsmedia/homebox) – Einfaches Inventar für Hardware und Gegenstände. 🔵 🏠
- [BookStack](https://github.com/BookStackApp/BookStack) – Wiki für die Homelab-Doku (Notfallplan auch offline vorhalten!). 🔵 🏠
- [draw.io / diagrams.net](https://github.com/jgraph/drawio) – Netzpläne zeichnen. 🔵
- [Excalidraw](https://github.com/excalidraw/excalidraw) – Skizzenhafte Diagramme. 🟢

## Täuschung & Frühwarnung

- [Canarytokens](https://canarytokens.org) – Kostenlose Köder-Dateien/-URLs/-Zugangsdaten, die beim Öffnen alarmieren. 🟠
- [OpenCanary](https://github.com/thinkst/opencanary) – Leichtgewichtiger Honeypot fürs interne Netz. 🔴 🏠
- [T-Pot](https://github.com/telekom-security/tpotce) – Honeypot-Plattform der Telekom Security (nicht im Heimnetz exponieren, nur isoliert!). 🔴 🏠

## Strom & Hardware

- [NUT (Network UPS Tools)](https://github.com/networkupstools/nut) – USV-Überwachung und automatisches Herunterfahren. 🔵
- [PeaNUT](https://github.com/Brandawg93/PeaNUT) – Weboberfläche für NUT. 🔵 🏠

## Wissen & Referenzen

- [BSI IT-Grundschutz-Kompendium](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/IT-Grundschutz-Kompendium/it-grundschutz-kompendium_node.html) – Deutsche Referenz mit Bausteinen für Server, Netze, Container.
- [BSI für Bürger: Router absichern](https://www.bsi.bund.de/DE/Themen/Verbraucherinnen-und-Verbraucher/verbraucherinnen-und-verbraucher_node.html) – Verbraucherinfos zu Router, WLAN, Smart Home.
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks) – Härtungsleitfäden für Betriebssysteme, Docker, Browser u. v. m.
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html) – Kompakte Container-Härtung.
- [CISA Known Exploited Vulnerabilities](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) – Aktiv ausgenutzte Lücken: zuerst patchen.
- [Have I Been Pwned](https://haveibeenpwned.com) – Wurden deine Zugangsdaten geleakt?
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org) – Sichere TLS-Konfigurationen für Webserver.
- [r/homelab](https://www.reddit.com/r/homelab/) und [r/selfhosted](https://www.reddit.com/r/selfhosted/) – Communities.

## Andere Awesome-Listen

- [awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted) – Die große Liste selbst hostbarer Software.
- [awesome-security](https://github.com/sbilly/awesome-security) – Allgemeine Security-Tools.
- [awesome-sysadmin](https://github.com/awesome-foss/awesome-sysadmin) – Werkzeuge für Systemadministration.
- [awesome-docker](https://github.com/veggiemonk/awesome-docker) – Alles rund um Docker.
- [awesome-incident-response](https://github.com/meirwah/awesome-incident-response) – Werkzeuge für Incident Response.

---

Fehlt ein Tool? → [CONTRIBUTING.md](CONTRIBUTING.md)
