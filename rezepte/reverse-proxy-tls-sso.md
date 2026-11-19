# Reverse Proxy, TLS & SSO

**Level:** 🟠 Medium · **Zeit:** 2–4 h · **Checkliste:** MED-NET-07, MED-ZUG-03, MED-IAM-01/02

## Warum?

- **Ein** Einstiegspunkt statt vieler offener Ports
- **HTTPS** mit gültigen Zertifikaten – auch intern. Wer sich an Zertifikatswarnungen gewöhnt, erkennt einen echten Angriff nicht mehr.
- **Single Sign-On mit 2FA** vor Diensten, die selbst keine gute Anmeldung haben

## Stufe KISS: `tailscale serve`

Wenn du ohnehin Tailscale nutzt: In der Admin-Konsole *MagicDNS* und *HTTPS Certificates* aktivieren, dann auf dem Host:

```bash
sudo tailscale serve --bg 8080
# → https://<hostname>.<tailnet>.ts.net mit gültigem Let's-Encrypt-Zertifikat
sudo tailscale serve status
```

Kein Reverse Proxy, keine Domain, keine Konfiguration. Nur im Tailnet erreichbar.

> ℹ️ Hostnamen mit Zertifikat landen in öffentlichen **Certificate-Transparency-Logs**. Der Tailnet-Name wird dadurch sichtbar – die Dienste selbst bleiben unerreichbar.

## Stufe Medium: Caddy mit eigener Domain

### 1. Domain & DNS

- Eine eigene Domain, z. B. `example.com`; interne Dienste unter `*.home.example.com`.
- **Intern** zeigen die Namen auf die LAN- oder Tailscale-IP des Reverse Proxys – per lokalem DNS (Pi-hole, AdGuard Home, Unbound, Router) oder einem öffentlichen A-Record auf eine private IP.
- Zertifikate per **DNS-01-Challenge** – funktioniert ohne offenen Port. Als **Wildcard** (`*.home.example.com`), damit nicht jeder Dienstname in den CT-Logs auftaucht.

### 2. Caddy mit DNS-Plugin

Caddy braucht für DNS-01 ein Plugin für deinen DNS-Anbieter (Cloudflare, Hetzner, deSEC, INWX, …). Eigenes Image bauen:

```dockerfile
# Dockerfile
FROM caddy:2.8.4-builder AS builder
RUN xcaddy build --with github.com/caddy-dns/cloudflare

FROM caddy:2.8.4
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

```yaml
services:
  caddy:
    build: .
    ports:
      - "192.168.10.5:443:443"      # LAN-IP des Hosts
      - "100.64.12.34:443:443"      # Tailscale-IP
    environment:
      - CF_API_TOKEN=${CF_API_TOKEN}   # Token nur mit Zone:DNS:Edit für diese Zone!
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy-data:/data
    networks: [web]
    restart: unless-stopped
volumes: { caddy-data: {} }
networks: { web: {} }
```

`Caddyfile`:

```caddyfile
*.home.example.com {
    tls {
        dns cloudflare {env.CF_API_TOKEN}
    }

    @grafana host grafana.home.example.com
    handle @grafana {
        reverse_proxy grafana:3000
    }

    @kuma host status.home.example.com
    handle @kuma {
        reverse_proxy uptime-kuma:3001
    }

    handle {
        abort
    }
}
```

Die Backend-Container hängen im Netzwerk `web` und veröffentlichen **keine** eigenen Ports.

Alternativen: **Traefik** (Konfiguration über Docker-Labels), **Nginx Proxy Manager** (Weboberfläche, KISS-freundlich).

### 3. SSO mit Authelia (Forward Auth)

Authelia sitzt vor den Diensten: Erst Login mit Passwort + 2FA (TOTP oder WebAuthn/Passkey), dann Weiterleitung.

Ergänzung im `Caddyfile` für einen geschützten Dienst:

```caddyfile
    @app host app.home.example.com
    handle @app {
        forward_auth authelia:9091 {
            uri /api/authz/forward-auth
            copy_headers Remote-User Remote-Groups Remote-Email Remote-Name
        }
        reverse_proxy app:8080
    }
```

Authelia selbst braucht eine Konfiguration (Benutzer-Datei oder LDAP, Session-Domain, Secrets) – siehe die offizielle Doku unter *Integration → Proxies → Caddy*. Alternativen:

- **Authentik** – mehr Funktionen (OIDC, SAML, LDAP-Outpost, Weboberfläche), mehr Ressourcen
- **Pocket ID** – schlanker OIDC-Provider nur mit Passkeys
- **Keycloak** – Enterprise-Standard, für Homelab oft überdimensioniert

Wo Dienste **OIDC** selbst unterstützen (Nextcloud, Grafana, Proxmox, Immich, Gitea/Forgejo …), lieber direkt per OIDC anbinden statt per Forward Auth.

## Öffentlich erreichbar machen (wenn es sein muss)

Zusätzlich zu allem oben:

- [ ] Nur Port **443** freigeben, nur auf den Reverse Proxy.
- [ ] Nur explizit benannte Hosts ausliefern, alles andere `abort` (siehe Caddyfile).
- [ ] SSO/2FA vor jedem Dienst, der nicht für anonyme Nutzung gedacht ist.
- [ ] **CrowdSec** mit Bouncer am Proxy → [Fail2ban & CrowdSec](fail2ban-crowdsec.md).
- [ ] Reverse Proxy und exponierte Dienste in einem eigenen Netz/VLAN (DMZ) → [Netzwerk segmentieren](netzwerk-segmentierung.md).
- [ ] Optional: Geo-Blocking, Rate-Limiting.
- [ ] Admin-Pfade (`/admin`, `/wp-admin` …) nur intern.

## Prüfen

- [ ] Alle internen Dienste per `https://` ohne Zertifikatswarnung erreichbar.
- [ ] Backend-Ports sind direkt **nicht** erreichbar (`nmap` gegen den Host).
- [ ] Geschützter Dienst fragt im privaten Browserfenster nach Login + 2FA.
- [ ] `curl -sI https://unbekannt.home.example.com` liefert keine Inhalte.

## Fallstricke

- **API-Token zu mächtig:** Der DNS-Token sollte nur DNS-Einträge einer Zone ändern dürfen – er kann sonst deine ganze Domain übernehmen.
- **DNS-Rebind-Schutz** des Routers (z. B. FRITZ!Box) blockiert öffentliche DNS-Namen, die auf private IPs zeigen. Domain als Ausnahme eintragen oder lokalen DNS nutzen.
- **Forward Auth und Apps mit eigener API/Apps** (Mobil-Apps, WebDAV, Jellyfin-Clients) vertragen sich oft nicht. Pfade gezielt ausnehmen oder OIDC nutzen.
- **Header-Vertrauen:** Dienste, die `Remote-User` vertrauen, dürfen **nur** über den Proxy erreichbar sein – sonst kann jeder den Header selbst setzen.
