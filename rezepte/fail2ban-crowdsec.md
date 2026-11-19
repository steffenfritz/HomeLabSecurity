# Fail2ban & CrowdSec

**Level:** 🔵 Low · **Zeit:** 20 min · **Checkliste:** LOW-SYS-05, MED-ZUG-03

## Warum?

Nur relevant, wenn etwas **öffentlich erreichbar** ist (Reverse Proxy, Mailserver, SSH auf einem VPS). Beide Tools lesen Logs, erkennen Brute-Force- und Scan-Muster und sperren die angreifende IP per Firewall.

> 💡 **KISS:** Wenn nichts öffentlich erreichbar ist ([Mesh-VPN](ports-schliessen-mesh-vpn.md)), brauchst du keins von beiden.

| | Fail2ban | CrowdSec |
|---|---|---|
| Prinzip | Lokale Regeln, lokale Sperren | Lokale Erkennung + **Community-Blocklisten** |
| Konfiguration | `jail.local`, Regex-Filter | Collections per `cscli` |
| Reverse-Proxy-Integration | über Logdateien | Bouncer für Traefik, Caddy, nginx, OPNsense, Cloudflare … |
| Datenschutz | nichts verlässt den Host | angreifende IPs werden (anonymisiert) geteilt; abschaltbar |

## Variante A: Fail2ban (minimal, für SSH)

```bash
sudo apt install fail2ban
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
bantime  = 1h
findtime = 10m
maxretry = 5
# eigene Netze nie sperren
ignoreip = 127.0.0.1/8 ::1 192.168.10.0/24 100.64.0.0/10

[sshd]
enabled = true
# Debian 12+ hat standardmäßig kein /var/log/auth.log:
backend = systemd
```

```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

## Variante B: CrowdSec

```bash
# Repository einrichten (Skript vorher lesen)
curl -s https://install.crowdsec.net | sudo sh
sudo apt install crowdsec
# Bouncer = setzt die Sperren um, hier per nftables/iptables
sudo apt install crowdsec-firewall-bouncer-nftables   # oder ...-iptables
```

CrowdSec erkennt installierte Dienste meist selbst. Weitere Collections, z. B. für Caddy oder Nginx Proxy Manager:

```bash
sudo cscli collections list
sudo cscli collections install crowdsecurity/caddy
sudo systemctl reload crowdsec
```

Wichtige Befehle:

```bash
sudo cscli metrics                 # was wird gelesen, was erkannt?
sudo cscli alerts list
sudo cscli decisions list          # aktuelle Sperren
sudo cscli decisions delete --ip 1.2.3.4   # versehentliche Sperre aufheben
```

Eigenes Netz freistellen: Allowlist anlegen (`cscli allowlists`, ab neueren Versionen) oder per Parser-Whitelist `/etc/crowdsec/parsers/s02-enrich/my-whitelist.yaml`.

Läuft der Reverse Proxy in Docker, müssen dessen **Access-Logs** für CrowdSec lesbar sein (Volume in eine Datei schreiben und in `/etc/crowdsec/acquis.d/` eintragen).

## Prüfen

- [ ] Von einer externen IP (Handy-Hotspot) mehrfach falsch einloggen → IP wird gesperrt (`fail2ban-client status sshd` bzw. `cscli decisions list`).
- [ ] Sperre wieder aufheben – und merken, wie das geht.

## Fallstricke

- **Hinter einem Reverse Proxy oder Tunnel** sehen Dienste oft nur die IP des Proxys. Dann sperrst du dich selbst oder niemanden. Echte Client-IP über `X-Forwarded-For` / Proxy-Protocol weiterreichen und am Proxy sperren (Bouncer).
- **Nicht als einzige Maßnahme:** Fail2ban/CrowdSec ersetzen weder starke Passwörter noch 2FA noch Updates.
- Bei **Cloudflare Tunnel** muss die Sperre bei Cloudflare passieren (CrowdSec Cloudflare-Bouncer), nicht auf dem Host.
