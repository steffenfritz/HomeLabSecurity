# Ports schließen & Mesh-VPN verwenden

**Level:** 🟢 KISS · **Zeit:** 30–60 min · **Checkliste:** KISS-ZUG-01, KISS-ZUG-02, KISS-NET-02, LOW-ZUG-01…03

## Warum?

Jeder offene Port ist rund um die Uhr Ziel automatisierter Scans. Ein Mesh-VPN wie **Tailscale** oder **NetBird** baut verschlüsselte WireGuard-Verbindungen zwischen deinen Geräten auf – **ohne eine einzige Portfreigabe**. Deine Dienste sind von unterwegs erreichbar, für das Internet aber unsichtbar.

Das ist die mit Abstand wirksamste Einzelmaßnahme in diesem Repo.

## Voraussetzungen

- Zugang zur Router-Oberfläche
- Ein Konto bei Tailscale oder NetBird (oder eine eigene Headscale-/NetBird-Instanz, siehe unten)

## Schritte

### 1. Bestandsaufnahme: Was ist offen?

**Im Router:** Liste aller Portfreigaben anschauen und notieren. Bei der FRITZ!Box: *Internet → Freigaben → Portfreigaben*. Pro Gerät auch prüfen, ob *„Selbstständige Portfreigaben für dieses Gerät erlauben“* aktiv ist.

**Auf jedem Host:** Welche Dienste lauschen auf welchen Adressen?

```bash
sudo ss -tulpn
```

Alles, was auf `0.0.0.0` oder `[::]` lauscht, ist potenziell von außen erreichbar.

**Von außen:** Von einem Gerät außerhalb deines Netzes (Handy-Hotspot, VPS):

```bash
# Öffentliche IP ermitteln (von zu Hause aus)
curl -4 https://ifconfig.me ; curl -6 https://ifconfig.me

# Von außerhalb scannen
nmap -Pn -p- <deine-öffentliche-IPv4>
nmap -6 -Pn -p- <IPv6-Adresse-deines-Servers>
```

Zusätzlich: `https://www.shodan.io/host/<deine-IP>` zeigt, was Suchmaschinen für IoT bereits über dich wissen.

> ⚠️ **IPv6 nicht vergessen.** Mit IPv6 hat jeder Host eine global erreichbare Adresse. Eine Portfreigabe für IPv6 öffnet den Port direkt am Gerät.

### 2. Mesh-VPN installieren

**Tailscale** (auf jedem Gerät: Server, Laptop, Handy):

```bash
# Skript vorher lesen: https://tailscale.com/install.sh
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

**NetBird** (Alternative, Open Source inkl. Control-Plane):

```bash
# Skript vorher lesen: https://pkgs.netbird.io/install.sh
curl -fsSL https://pkgs.netbird.io/install.sh | sh
netbird up
# oder für Server ohne Browser:
netbird up --setup-key <SETUP-KEY>
```

Für Handys und Desktops gibt es Apps in den jeweiligen Stores.

**Ganzes Heimnetz erreichbar machen (Subnet Router)** – praktisch für Geräte, auf denen kein Client läuft (NAS, Drucker, Router-UI):

```bash
# IP-Forwarding aktivieren
echo 'net.ipv4.ip_forward = 1' | sudo tee /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf

sudo tailscale up --advertise-routes=192.168.10.0/24
```

Danach die Route in der Admin-Konsole freigeben. NetBird löst das über *Network Routes* in der Weboberfläche.

> 💡 **KISS-Alternative ohne Zusatzsoftware:** Viele Router haben ein eingebautes VPN. Die FRITZ!Box kann seit FRITZ!OS 7.50 WireGuard. Das öffnet zwar einen UDP-Port, WireGuard antwortet aber nicht auf unauthentifizierte Pakete und ist für Scanner praktisch unsichtbar.

### 3. Dienste an die richtigen Adressen binden

Ein Dienst, der nur über VPN oder lokal erreichbar sein soll, sollte auch nur dort lauschen. In `docker-compose.yml`:

```yaml
services:
  app:
    ports:
      - "127.0.0.1:8080:80"        # nur lokal (z. B. hinter Reverse Proxy)
      # - "100.64.12.34:8080:80"   # nur über die Tailscale-IP dieses Hosts
      # - "192.168.10.5:8080:80"   # nur im LAN
```

Statt `- "8080:80"` – das bindet an **alle** Interfaces. Warum das besonders wichtig ist: [Docker umgeht ufw](host-firewall.md#docker-umgeht-ufw).

### 4. Zugriffsregeln (ACLs) setzen

Standardmäßig darf in einem neuen Tailnet jedes Gerät jedes andere erreichen. Besser: Server taggen und Zugriff einschränken. Beispiel für die Tailscale-Policy (*Admin Console → Access Controls*):

```jsonc
{
  "tagOwners": {
    "tag:server": ["autogroup:admin"]
  },
  "acls": [
    // Admins: SSH und Web auf alle Server
    {"action": "accept", "src": ["autogroup:admin"],  "dst": ["tag:server:22,443"]},
    // Familie: nur Web
    {"action": "accept", "src": ["autogroup:member"], "dst": ["tag:server:443"]}
  ]
}
```

Server taggen: `sudo tailscale up --advertise-tags=tag:server`. Bei NetBird entspricht das *Groups* und *Access Control Policies*; die Default-Policy „All → All“ solltest du durch eigene Regeln ersetzen.

**Key-Expiry:** Für Clients aktiv lassen. Für Server, die immer erreichbar sein müssen, bewusst deaktivieren.

### 5. Portfreigaben entfernen

Jetzt, wo alles über das VPN geht: Alle Portfreigaben im Router löschen, **UPnP deaktivieren**.

## Prüfen

- [ ] `nmap` von außen (Schritt 1) zeigt keine offenen Ports mehr – IPv4 und IPv6.
- [ ] Vom Handy im Mobilfunknetz mit aktivem VPN: Dienste erreichbar.
- [ ] Vom Handy im Mobilfunknetz **ohne** VPN: Dienste **nicht** erreichbar.

## Was, wenn etwas öffentlich sein muss?

Zum Beispiel Jellyfin für Verwandte ohne VPN-App, oder ein Webhook-Empfänger. Optionen, von einfach nach aufwändig:

| Option | Vorteil | Nachteil |
|---|---|---|
| Andere Person ins Tailnet einladen / Node teilen | Kein Port offen | Person braucht App |
| **Tailscale Funnel** | Kein Port offen, TLS automatisch | Traffic über Tailscale-Infrastruktur, eingeschränkte Ports |
| **Cloudflare Tunnel** | Kein Port offen, WAF/DDoS-Schutz | Cloudflare terminiert TLS und sieht den Klartext; ToS für Video-Streaming beachten |
| **Pangolin** auf eigenem VPS | Selbst gehostet, kein Port zu Hause offen | Ein VPS mehr zu warten |
| Portfreigabe **nur 443** auf Reverse Proxy | Volle Kontrolle | Du bist voll verantwortlich: SSO, CrowdSec, Updates → [Reverse Proxy, TLS & SSO](reverse-proxy-tls-sso.md) |

Niemals öffentlich: SSH, SMB, RDP, Datenbanken, Proxmox/NAS-Admin, IPMI, Docker-API, Home-Assistant ohne starke Authentifizierung.

## Fallstricke

- **Abhängigkeit vom Anbieter:** Fällt die Tailscale-Koordination aus, funktionieren bestehende Verbindungen meist weiter, neue Logins aber nicht. Plane einen **Notfallzugang** (lokaler Zugriff, Router-VPN) → [Notfallplan](notfallplan.md).
- **Selbst hosten?** Headscale bzw. eine eigene NetBird-Instanz geben dir volle Kontrolle – aber die Control-Plane muss dann öffentlich erreichbar sein und von dir gepatcht werden. KISS-Empfehlung: Mit der gehosteten Variante starten.
- **Subnet-Konflikte:** Wenn dein Heimnetz `192.168.0.0/24` oder `192.168.1.0/24` ist, kollidiert es oft mit Hotel-/Freunde-WLANs. Ein ungewöhnlicheres Subnetz (z. B. `10.42.10.0/24`) spart Ärger.
- **Tailscale SSH** (`tailscale up --ssh`) ist bequem, ersetzt aber die SSH-Authentifizierung durch die Tailscale-Identität. Dann ist 2FA beim Tailscale-Login Pflicht.

## Weiter

- [Host-Firewall](host-firewall.md) – damit auch im LAN nur das erreichbar ist, was soll.
- [Reverse Proxy, TLS & SSO](reverse-proxy-tls-sso.md) – schöne HTTPS-Namen statt `http://100.x.y.z:8080`.
