# Monitoring

**Level:** 🟢 KISS → 🟠 Medium · **Zeit:** 30 min (KISS) bis 1 Tag (Medium) · **Checkliste:** KISS-MON-01, LOW-MON-01/02, MED-MON-01…04

## Warum?

Sicherheit heißt auch **Verfügbarkeit** und **Integrität**: Eine volllaufende Platte, ein sterbendes RAID, ein seit Wochen fehlgeschlagenes Backup oder ein abgelaufenes Zertifikat fallen ohne Monitoring erst auf, wenn es zu spät ist. Und ungewöhnliche Last kann ein Hinweis auf einen Kryptominer sein.

## Stufe KISS: Uptime Kuma + ntfy

**Uptime Kuma** prüft, ob Dienste erreichbar sind, und warnt per Push. **ntfy** bringt die Nachricht aufs Handy.

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1.23.16     # Beispielversion – aktuelle prüfen
    volumes:
      - ./data:/app/data
    ports:
      - "127.0.0.1:3001:3001"               # oder Tailscale-IP
    restart: unless-stopped
```

1. `docker compose up -d`, dann `http://<host>:3001` öffnen und Admin-Konto anlegen.
2. ntfy-App aufs Handy, ein **langes, zufälliges Topic** abonnieren (auf ntfy.sh ist jedes Topic öffentlich, wer den Namen kennt). Oder ntfy selbst hosten.
3. In Uptime Kuma: *Einstellungen → Benachrichtigungen → ntfy* einrichten.
4. Monitore anlegen: HTTP(S) für jede Web-App, Ping für Router/NAS, TCP-Port für Datenbanken, **Zertifikatsablauf** aktivieren.

> 💡 Uptime Kuma auf einem Host laufen lassen, der nicht der wichtigste Server ist – oder zusätzlich einen externen Dienst (z. B. **Healthchecks.io**) nutzen, der meldet, wenn Uptime Kuma selbst schweigt.

## Stufe Low: Was zusätzlich überwacht werden sollte

| Was | Warum | Wie (einfach) |
|---|---|---|
| Plattenplatz | Volle Platte = kaputte DBs, keine Logs | Checkmk/Beszel/Netdata, oder Skript + Healthchecks |
| **SMART** | Platten kündigen ihren Tod oft an | `smartmontools` mit `smartd`, Mail/ntfy bei Fehlern |
| **RAID / ZFS** | Degradiertes Array unbemerkt = nächster Ausfall ist Datenverlust | `zed` (ZFS Event Daemon), `mdadm --monitor`, NAS-Benachrichtigungen |
| ZFS Scrubs | Bit-Rot erkennen | monatlicher Scrub (bei Proxmox/TrueNAS Standard) |
| **Backups** | Siehe [Backup-Rezept](backup-3-2-1.md) | Healthchecks-Ping |
| Zertifikate | Abgelaufen = Warnungen, die man „wegklickt“ | Uptime Kuma |
| Ausstehende Updates | Patch-Stand | Checkmk / Prometheus node_exporter |
| USV | Akku tot = kein sauberes Herunterfahren | NUT + Monitoring |
| Temperatur | Lüfterausfall | Checkmk / Netdata |

SMART schnell aktiv:

```bash
sudo apt install smartmontools
```

In `/etc/smartd.conf` die `DEVICESCAN`-Zeile ersetzen – alle Platten, Kurztest täglich 2 Uhr, Langtest samstags 3 Uhr, Mail an root:

```text
DEVICESCAN -a -o on -S on -s (S/../.././02|L/../../6/03) -m root -M exec /usr/share/smartmontools/smartd-runner
```

```bash
sudo systemctl restart smartmontools   # Dienstname je nach Distribution: smartd
```

Damit Mails an root ankommen, braucht der Host einen Mail-Relay (z. B. `msmtp-mta`) – oder du überlässt SMART-Warnungen deinem Monitoring-Tool.

## Stufe Medium: Zentrales Monitoring

Wähle **eins**. Mehrere Monitoring-Systeme parallel sind Wartungsaufwand ohne Mehrwert.

| Tool | Stärke | Aufwand | Passt, wenn … |
|---|---|---|---|
| **Beszel** | Sehr leichtgewichtig, schöne UI, Docker-Stats | gering | … du nur Hosts & Container sehen willst |
| **Checkmk** (Raw/Community Edition) | Autodiscovery, hunderte Checks out of the box, SNMP für Switches/NAS | mittel | … du viele unterschiedliche Geräte hast und wenig konfigurieren willst |
| **Zabbix** | Sehr mächtig, Templates für fast alles | mittel–hoch | … du Zabbix beruflich lernen willst |
| **Prometheus + Grafana** (+ node_exporter, Alertmanager) | Flexibel, Standard in Cloud-Native | hoch | … du gern Dashboards baust und Metriken abfragst |
| **Netdata** | Echtzeit, per-Sekunde-Metriken, Null-Konfiguration | gering | … du schnell tiefe Einblicke in einen Host willst |

### Checkmk – Kurzanleitung

1. Checkmk-Server installieren (Docker-Image oder Paket für Debian/Ubuntu; siehe [checkmk.com](https://checkmk.com) → Download → kostenlose Edition).
2. Site anlegen, im Webinterface anmelden, Admin-Passwort ändern.
3. Auf jedem Host den **Agenten** installieren (Download aus dem eigenen Checkmk-Server unter *Setup → Agents*) und für **TLS-Registrierung** registrieren:
   ```bash
   sudo cmk-agent-ctl register --hostname <host> --server <checkmk-server> --site <site> --user <user>
   ```
4. Host in Checkmk anlegen → *Service discovery* → Änderungen aktivieren.
5. Benachrichtigungen einrichten (E-Mail, ntfy per Plugin/Skript, Telegram, …).
6. Switches/NAS per **SNMP** (v3, wenn möglich) hinzufügen.

### Prometheus-Stack – Minimal

```yaml
services:
  prometheus:
    image: prom/prometheus:v3.0.1             # Beispielversion
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prom-data:/prometheus
    ports: ["127.0.0.1:9090:9090"]
  grafana:
    image: grafana/grafana:11.4.0             # Beispielversion
    volumes: [grafana-data:/var/lib/grafana]
    ports: ["127.0.0.1:3000:3000"]
volumes: { prom-data: {}, grafana-data: {} }
```

Auf jedem Host `prometheus-node-exporter` (Debian-Paket) installieren; Port `9100` per Firewall nur für den Prometheus-Host freigeben. Alerts über **Alertmanager**.

## Logs (Medium)

Zentrales Logging hilft bei Fehlersuche und Sicherheitsvorfällen:

- **KISS:** `journald` mit ausreichender Größe (`SystemMaxUse=2G` in `/etc/systemd/journald.conf`), **Dozzle** für Container-Logs.
- **Medium:** **Grafana Loki** + Alloy/Promtail, oder **Graylog**.
- **High:** Logs ins SIEM → [Wazuh](wazuh.md).

## Prüfen

- [ ] Einen Testdienst stoppen → Benachrichtigung kommt in < 5 min aufs Handy.
- [ ] Monitoring-Server stoppen → externer Dead-Man's-Switch meldet sich.
- [ ] Du bekommst **nicht** täglich Dutzende irrelevante Meldungen (sonst: Schwellen anpassen).

## Fallstricke

- **Alarmmüdigkeit:** Zu viele Meldungen führen dazu, dass die wichtige übersehen wird. Lieber wenige, gut gewählte Alarme.
- **Monitoring als Angriffsfläche:** Agenten laufen oft als root, Monitoring-Server kennen alle Hosts. Monitoring-UI nicht öffentlich, Agenten-Kommunikation verschlüsselt (Checkmk TLS-Registrierung, SNMPv3).
- **Monitoring auf demselben Host**, der ausfällt, meldet nichts.
