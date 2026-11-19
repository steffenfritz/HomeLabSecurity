# Automatische Updates & Patch-Tag

**Level:** 🟢 KISS · **Zeit:** 15 min · **Checkliste:** KISS-SYS-01, LOW-UPD-01…03, MED-SYS-02

## Warum?

Die meisten erfolgreichen Angriffe nutzen **bekannte** Lücken, für die es längst Patches gibt. Sicherheitsupdates des Betriebssystems sind risikoarm und sollten automatisch laufen. Alles andere (Container, Hypervisor, Firmware) bekommt einen festen Termin.

## Debian / Ubuntu: unattended-upgrades

```bash
sudo apt install unattended-upgrades apt-listchanges
sudo dpkg-reconfigure -plow unattended-upgrades    # "Ja" wählen
```

Standardmäßig werden nur **Sicherheitsupdates** installiert. Eigene Einstellungen in eine separate Datei:

```bash
sudo nano /etc/apt/apt.conf.d/52unattended-upgrades-local
```

```text
// Nach Kernel-Updates nachts neu starten
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "04:00";

// Aufräumen
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
```

Prüfen:

```bash
sudo unattended-upgrade --dry-run --debug
cat /var/log/unattended-upgrades/unattended-upgrades.log
```

> 💡 `Automatic-Reboot` nur auf Hosts, bei denen ein nächtlicher Neustart okay ist. Auf dem Hypervisor oder NAS stattdessen `needrestart` installieren und am Patch-Tag manuell neu starten.

## Fedora / RHEL / Alma / Rocky: dnf-automatic

```bash
sudo dnf install dnf-automatic
sudo nano /etc/dnf/automatic.conf
```

```ini
[commands]
upgrade_type = security
apply_updates = yes
```

```bash
sudo systemctl enable --now dnf-automatic.timer
```

Bei neueren Fedora-Versionen mit **dnf5** liegt die Konfiguration unter `/etc/dnf/dnf5-plugins/automatic.conf` (Vorlage unter `/usr/share/dnf5/dnf5-plugins/automatic.conf`).

## Proxmox VE

Proxmox **nicht** vollautomatisch aktualisieren. Am Patch-Tag:

```bash
apt update
apt full-upgrade      # nicht "apt upgrade" – Proxmox braucht full-upgrade/dist-upgrade
```

Vorher: laufende Backups abwarten, Release Notes bei Point-Releases lesen. Major-Upgrades (z. B. 8 → 9) nur nach der offiziellen Upgrade-Anleitung im Proxmox-Wiki und mit dem dort beschriebenen Prüfskript (z. B. `pve8to9`).

## Der Patch-Tag

Ein fester wiederkehrender Termin, z. B. **erster Sonntag im Monat**, 30–60 Minuten:

1. [ ] Router-/Firewall-Firmware (FRITZ!Box, OPNsense, OpenWrt)
2. [ ] Switch, Access Points
3. [ ] Hypervisor (Proxmox) und NAS
4. [ ] Container: Meldungen von Diun/Renovate abarbeiten → [Docker-Images pinnen](docker-images-pinnen.md)
5. [ ] Hosts mit ausstehendem Neustart neu starten (`needrestart -r l` bzw. `/var/run/reboot-required`)
6. [ ] Kurzer Funktionstest der wichtigsten Dienste
7. [ ] Geräte ohne Updates mehr (End of Life)? Ersetzen oder isolieren.

Kalendereintrag mit dieser Liste anlegen – fertig.

## Prüfen

- [ ] `/var/log/unattended-upgrades/` bzw. `journalctl -u dnf-automatic` zeigt erfolgreiche Läufe.
- [ ] Patch-Tag steht im Kalender.

## Fallstricke

- **Drittanbieter-Repos** (Docker, Tailscale, …) sind bei `unattended-upgrades` standardmäßig **nicht** dabei. Wer sie einschließen will, ergänzt `Unattended-Upgrade::Origins-Pattern`. KISS-Ansicht: Am Patch-Tag manuell ist auch okay.
- Wenn `apt` gerade läuft (unattended), blockiert es andere `apt`-Aufrufe. Kurz warten statt Lock-Dateien zu löschen.
- Automatische Updates ersetzen kein **Monitoring**: Ein gescheiterter Update-Lauf fällt nur auf, wenn jemand hinschaut → [Monitoring](monitoring.md).
