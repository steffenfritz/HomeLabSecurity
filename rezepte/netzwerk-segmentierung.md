# Netzwerk segmentieren

**Level:** 🟠 Medium · **Zeit:** 1 Tag · **Checkliste:** KISS-NET-04, MED-NET-01…06, HIGH-ZUG-03

## Warum?

In einem flachen Netz kann jedes Gerät jedes andere erreichen. Die billige Überwachungskamera mit Firmware von 2019 steht direkt neben deinem NAS mit allen Familienfotos. Segmentierung begrenzt den Schaden, wenn ein Gerät kompromittiert wird.

## Stufe KISS: Gäste-WLAN

Ohne neue Hardware: Gäste **und IoT-Geräte** ins Gäste-WLAN des Routers. Die FRITZ!Box trennt das Gästenetz vom Heimnetz; Geräte darin erreichen nur das Internet.

Geht nicht für Geräte, die mit Servern im LAN sprechen müssen (z. B. Home-Assistant-Integrationen, Chromecast) – dafür braucht es VLANs.

## Stufe Medium: VLANs

### Voraussetzungen

- Ein **Router/Firewall**, der VLANs und Inter-VLAN-Regeln kann: OPNsense, pfSense, OpenWrt, UniFi, MikroTik … (Die FRITZ!Box kann das nicht.)
- Ein **managed Switch** (VLAN-fähig, 802.1Q)
- **Access Points**, die mehrere SSIDs auf VLANs mappen können

### 1. VLAN-Plan

| VLAN | Name | Subnetz | Wer | Beispiel |
|---|---|---|---|---|
| 10 | MGMT | 10.42.10.0/24 | Infrastruktur-Oberflächen | Switch, APs, Proxmox-UI, IPMI/BMC, NAS-Admin |
| 20 | SERVER | 10.42.20.0/24 | Dienste | Docker-Hosts, NAS-Freigaben, VMs |
| 30 | CLIENTS | 10.42.30.0/24 | Vertrauenswürdige Geräte | Laptops, Handys der Familie |
| 40 | IOT | 10.42.40.0/24 | Nicht vertrauenswürdige Geräte | TV, Steckdosen, Saugroboter, Kameras |
| 50 | GUEST | 10.42.50.0/24 | Gäste | nur Internet |
| 60 | DMZ | 10.42.60.0/24 | Öffentlich erreichbare Dienste *(optional)* | Reverse Proxy, exponierte Apps |

> 💡 Ungewöhnliche Subnetze (`10.42.x.0/24` statt `192.168.1.0/24`) vermeiden Konflikte mit fremden Netzen, wenn du per VPN von unterwegs zugreifst.

### 2. Regel-Matrix

Grundsatz: **Alles verboten, was nicht explizit erlaubt ist.** Gelesen als „Von Zeile nach Spalte“:

| Von ↓ / Nach → | MGMT | SERVER | CLIENTS | IOT | Internet |
|---|---|---|---|---|---|
| **MGMT** | ✅ | ✅ | ❌ | ✅ (Verwaltung) | ✅ (Updates) |
| **SERVER** | ❌ | ✅ | ❌ | ⚠️ nur benötigte (z. B. Home Assistant → IoT) | ✅ |
| **CLIENTS** | ⚠️ nur Admin-Geräte | ⚠️ 443, ggf. SMB/NFS | ✅ | ⚠️ z. B. Casting | ✅ |
| **IOT** | ❌ | ⚠️ nur MQTT/HA-Port | ❌ | ❌ (Client Isolation) | ⚠️ nur wo nötig |
| **GUEST** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **DMZ** | ❌ | ⚠️ einzelne Backends | ❌ | ❌ | ✅ |

- Antworten auf erlaubte Verbindungen sind durch die Stateful Firewall automatisch erlaubt.
- **Jedes** VLAN darf den eigenen DNS-Resolver (Port 53) und DHCP auf dem Gateway erreichen.
- **Firewall-Oberfläche** (OPNsense-UI, SSH) nur aus MGMT erreichbar, nicht aus jedem VLAN über die Gateway-IP!

### 3. Umsetzung (am Beispiel OPNsense)

1. *Interfaces → Other Types → VLAN:* VLANs auf dem LAN-Interface anlegen.
2. *Interfaces → Assignments:* VLANs zuweisen, aktivieren, statische IP (Gateway, z. B. `10.42.40.1/24`).
3. *Services → DHCP (Kea/Dnsmasq/ISC):* DHCP pro VLAN.
4. *Firewall → Aliases:* `RFC1918` (alle privaten Netze), `HA_Server`, …
5. *Firewall → Rules → [VLAN]:* Regeln von oben nach unten, z. B. für IOT:
   ```text
   PASS   IOT net → IOT address  port 53 (DNS), 67 (DHCP)
   PASS   IOT net → HA_Server    port 1883 (MQTT)
   BLOCK  IOT net → RFC1918      any
   PASS   IOT net → any          any        # Internet (oder gezielter)
   ```
6. **DNS erzwingen** (Port Forward): Alles von IOT auf Port 53, das nicht zum eigenen Resolver geht, auf den eigenen Resolver umleiten.
7. Switch: Ports als **Access-Port** (untagged) im jeweiligen VLAN, Uplinks zu Router/APs/Hypervisor als **Trunk** (tagged).
8. APs: SSID pro VLAN (z. B. `Zuhause` → 30, `Zuhause-IoT` → 40, `Gast` → 50), **Client Isolation** für IoT und Gast.
9. Proxmox: Bridge *VLAN aware* aktivieren, VM-Netzwerkkarten mit VLAN-Tag versehen.

> ⚠️ **Nicht aussperren:** Zuerst MGMT-VLAN anlegen und testen, dass du von dort alles erreichst. Erst dann alte Zugänge entfernen. Einen Switch-Port und eine Notfall-IP im MGMT-Netz für den Laptop dokumentieren.

## Prüfen

Von einem Gerät in jedem VLAN:

```bash
nmap -sn 10.42.10.0/24        # Ping-Scan in andere VLANs → sollte (fast) nichts finden
nmap -p 22,443,8006 10.42.20.5
nslookup example.com 8.8.8.8   # aus IOT: wird umgeleitet / blockiert?
```

- [ ] IOT erreicht kein CLIENTS/SERVER/MGMT (außer explizit erlaubt).
- [ ] GUEST erreicht nur das Internet.
- [ ] Router-/Firewall-UI nur aus MGMT erreichbar.

## Fallstricke

- **mDNS/Chromecast/AirPlay** funktioniert nicht über VLAN-Grenzen. Lösung: mDNS-Repeater (OPNsense-Plugin `mdns-repeater`, Avahi-Reflector) – öffnet aber Löcher, bewusst einsetzen.
- **Home Assistant** braucht oft Zugriff auf IoT – HA ins SERVER-VLAN, gezielte Regeln Richtung IOT.
- **IPv6** pro VLAN mitdenken: Regeln gelten nicht automatisch für beide Protokolle.
- Ein falsch konfigurierter Trunk-Port kann VLANs „verbinden“. Native/Untagged-VLAN auf Trunks bewusst setzen (nicht VLAN 1 für Produktivverkehr).
