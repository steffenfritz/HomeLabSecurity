# Rezepte

Kurze, konkrete Anleitungen. Jedes Rezept nennt Level, Zeitaufwand, Schritte, einen **Prüf-Schritt** und typische **Fallstricke**.

| Rezept | Level | Zeit | Worum geht's? |
|---|---|---|---|
| [Ports schließen & Mesh-VPN](ports-schliessen-mesh-vpn.md) | 🟢 KISS | 30–60 min | Portfreigaben weg, Zugriff per Tailscale/NetBird |
| [Docker-Images pinnen](docker-images-pinnen.md) | 🟢 KISS → 🟠 | 15–60 min | Kein `:latest`, Digests, Diun, Renovate |
| [SSH härten](ssh-haerten.md) | 🟢 KISS → 🔴 | 15 min | Nur Schlüssel, kein Root, FIDO2 |
| [Automatische Updates](automatische-updates.md) | 🟢 KISS | 15 min | unattended-upgrades, dnf-automatic, Patch-Tag |
| [Passwörter, 2FA & Secrets](passwoerter-2fa-secrets.md) | 🟢 KISS → 🟠 | 1–2 h | Passwortmanager, 2FA-Prioritäten, `.env`, SOPS |
| [Backups nach 3-2-1](backup-3-2-1.md) | 🟢 KISS → 🟠 | 1–2 h | restic, DB-Dumps, Offsite, append-only, Restore-Test |
| [Monitoring](monitoring.md) | 🟢 KISS → 🟠 | 30 min – 1 Tag | Uptime Kuma + ntfy, Checkmk, Prometheus |
| [Host-Firewall](host-firewall.md) | 🔵 Low | 20 min | ufw, Docker-Falle, Proxmox-Firewall |
| [Container härten](container-haerten.md) | 🔵 Low → 🟠 | 30 min/Stack | Non-Root, Capabilities, Socket-Proxy |
| [Fail2ban & CrowdSec](fail2ban-crowdsec.md) | 🔵 Low | 20 min | Brute-Force abwehren, wenn etwas exponiert ist |
| [Notfallplan](notfallplan.md) | 🔵 Low → 🔴 | 1 h | Notfallzettel, was tun bei Kompromittierung |
| [Reverse Proxy, TLS & SSO](reverse-proxy-tls-sso.md) | 🟠 Medium | 2–4 h | Caddy, `tailscale serve`, Wildcard-Zertifikate, Authelia |
| [Netzwerk segmentieren](netzwerk-segmentierung.md) | 🟠 Medium | 1 Tag | VLAN-Plan, Regel-Matrix, IoT einsperren |
| [Audits & Scans](audits-und-scans.md) | 🟠 Medium → 🔴 | 1–2 h | Lynis, nmap, Trivy, docker-bench, Greenbone |
| [Wazuh einrichten](wazuh.md) | 🔴 High | 1 Tag | SIEM/XDR im Homelab: Single-Node, Agenten, FIM |

## Rezept-Vorlage

Neue Rezepte bitte nach diesem Schema (siehe auch [CONTRIBUTING.md](../CONTRIBUTING.md)):

```markdown
# Titel

**Level:** 🔵 Low · **Zeit:** 20 min · **Checkliste:** LOW-XXX-01

## Warum?
## Voraussetzungen
## Schritte
## Prüfen
## Fallstricke
## Weiter
```
