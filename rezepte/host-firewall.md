# Host-Firewall

**Level:** 🔵 Low · **Zeit:** 20 min · **Checkliste:** LOW-NET-01, LOW-NET-02, MED-SYS-03

## Warum?

Die Router-Firewall schützt vor dem Internet – aber nicht vor dem kompromittierten Smart-TV im selben Netz. Eine Host-Firewall sorgt dafür, dass jeder Server nur das anbietet, was er soll.

> ⚠️ SSH-Sitzung offen lassen und **zuerst SSH erlauben**, dann aktivieren.

## Schritte (ufw, Debian/Ubuntu)

```bash
sudo apt install ufw

sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH nur aus dem LAN und über Tailscale
sudo ufw allow from 192.168.10.0/24 to any port 22 proto tcp
sudo ufw allow in on tailscale0

# Web-Dienste (z. B. Reverse Proxy) aus dem LAN
sudo ufw allow from 192.168.10.0/24 to any port 443 proto tcp

sudo ufw enable
sudo ufw status verbose
```

Bei NetBird heißt das Interface `wt0`.

Fedora/RHEL nutzen **firewalld**:

```bash
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --permanent --zone=public --remove-service=cockpit   # Beispiel
sudo firewall-cmd --permanent --zone=trusted --add-interface=tailscale0
sudo firewall-cmd --reload
```

## Docker umgeht ufw

**Die wichtigste Falle in diesem Repo.** Docker schreibt für veröffentlichte Ports (`ports: - "8080:80"`) eigene iptables-/nftables-Regeln, die **vor** den ufw-Regeln greifen. Ein `ufw deny 8080` wirkt dann **nicht** – der Port ist trotzdem erreichbar.

Lösungen, von KISS nach aufwändig:

**1. An eine bestimmte IP binden (empfohlen):**

```yaml
ports:
  - "127.0.0.1:8080:80"      # nur lokal, z. B. für Reverse Proxy auf demselben Host
  - "100.64.12.34:8080:80"   # nur über Tailscale
```

**2. Standard-Bind-Adresse für Docker ändern** – dann bindet `- "8080:80"` automatisch nur an localhost. In `/etc/docker/daemon.json`:

```json
{
  "ip": "127.0.0.1"
}
```

```bash
sudo systemctl restart docker
```

**3. Gar keine Ports veröffentlichen:** Container hängen nur in einem gemeinsamen Docker-Netzwerk mit dem Reverse Proxy, der als Einziger Ports veröffentlicht.

**4. Regeln in der `DOCKER-USER`-Chain** oder Hilfsprojekte wie `ufw-docker` – funktioniert, ist aber fehleranfälliger.

Prüfen:

```bash
sudo ss -tlnp | grep docker-proxy   # oder: docker ps --format '{{.Names}}\t{{.Ports}}'
# Alles mit 0.0.0.0 oder [::] ist auf allen Interfaces erreichbar
```

## Proxmox

Proxmox hat eine eingebaute Firewall auf drei Ebenen (*Datacenter → Node → VM/CT*). Sie ist standardmäßig **aus**.

1. *Datacenter → Firewall → Options:* erst Regeln anlegen (Web-UI 8006 und SSH 22 aus dem Management-Netz erlauben), **dann** aktivieren.
2. Pro VM/CT: *Firewall → Options → Firewall: Yes*, und an der Netzwerkkarte der VM *Firewall* aktivieren.
3. Policy *Input: DROP* für VMs, die nur wenige Ports brauchen.

## Prüfen

- [ ] Von einem anderen Gerät im LAN: `nmap -p- <server-ip>` zeigt nur die gewollten Ports.
- [ ] `ufw status verbose` / `firewall-cmd --list-all` passt zur Dienste-Liste.
- [ ] Nach einem Neustart ist die Firewall weiterhin aktiv.

## Fallstricke

- **IPv6:** `ufw` regelt IPv6 mit, wenn `IPV6=yes` in `/etc/default/ufw` steht (Standard). Bei handgeschriebenen iptables-Regeln `ip6tables` nicht vergessen.
- **Tailscale/NetBird** fügen eigene Regeln hinzu – ufw-Regeln für das VPN-Interface trotzdem setzen, um die Absicht klar zu machen.
- **Ausgehend erlauben** ist bei Low okay. Egress-Filter für Server sind Thema in [High](../checkliste/03-high.md).
