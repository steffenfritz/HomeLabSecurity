# 🟢 Level KISS – Der Nachmittag

**Ziel:** Die größten Risiken mit minimalem Aufwand beseitigen. Keine neue Infrastruktur, keine neuen Dienste, die du warten musst (außer einem VPN-Client und einem Uptime-Check).

**Aufwand:** 2–4 Stunden · **Wartung:** praktisch keine

---

## Zugang von außen

- [ ] **KISS-ZUG-01** – **Keine Portfreigaben im Router.** Entferne alle Weiterleitungen, die du nicht bewusst und begründet brauchst.
  → [Rezept: Ports schließen & Mesh-VPN](../rezepte/ports-schliessen-mesh-vpn.md)
- [ ] **KISS-ZUG-02** – Fernzugriff ausschließlich über ein **Mesh-VPN** (Tailscale, NetBird) oder das VPN des Routers (z. B. WireGuard in der FRITZ!Box).
  → [Rezept: Ports schließen & Mesh-VPN](../rezepte/ports-schliessen-mesh-vpn.md)

## Netzwerk & Router

- [ ] **KISS-NET-01** – Router-Firmware aktuell, automatische Updates aktiviert.
- [ ] **KISS-NET-02** – **UPnP / „selbstständige Portfreigaben“ deaktiviert.** Sonst öffnen Geräte und Container still Ports.
- [ ] **KISS-NET-03** – WPS aus, Fernwartung des Routers aus dem Internet aus, Standard-Admin-Passwort geändert.
- [ ] **KISS-NET-04** – **Gäste-WLAN** für Gäste *und* für IoT-Geräte (Smart-TV, Steckdosen, Staubsauger …). Das ist die KISS-Variante der Netzsegmentierung.

## Konten & Passwörter

- [ ] **KISS-IAM-01** – **Passwortmanager** benutzen. Jedes Gerät und jeder Dienst hat ein eigenes, zufälliges Passwort. Alle Standardpasswörter sind geändert.
  → [Rezept: Passwörter, 2FA & Secrets](../rezepte/passwoerter-2fa-secrets.md)
- [ ] **KISS-IAM-02** – **2FA** für die kritischen Konten: E-Mail, Passwortmanager, VPN-Konto (Tailscale/NetBird-Login), Domain-Registrar, Cloud-Backup.

## Server & Container

- [ ] **KISS-SYS-01** – **Automatische Sicherheitsupdates** für das Betriebssystem aktiviert.
  → [Rezept: Automatische Updates](../rezepte/automatische-updates.md)
- [ ] **KISS-SYS-02** – **SSH nur mit Schlüssel**, Passwort-Login deaktiviert.
  → [Rezept: SSH härten](../rezepte/ssh-haerten.md)
- [ ] **KISS-CON-01** – **Docker-Images auf Versionen pinnen** – kein `:latest` in `docker-compose.yml`.
  → [Rezept: Docker-Images pinnen](../rezepte/docker-images-pinnen.md)
- [ ] **KISS-SYS-03** – Unbenutzte Dienste, Container, VMs und Testinstallationen **abschalten und löschen**.

## Backup

- [ ] **KISS-BAK-01** – Es gibt ein **automatisches Backup** der wichtigen Daten, davon **eine Kopie außer Haus** (Cloud, Freunde, Arbeit).
  → [Rezept: Backups nach 3-2-1](../rezepte/backup-3-2-1.md)
- [ ] **KISS-BAK-02** – **Einmal testweise wiederhergestellt.** Mindestens eine Datei, besser ein ganzer Dienst.
- [ ] **KISS-BAK-03** – Das Backup-Passwort / der Schlüssel liegt im Passwortmanager **und** zusätzlich offline (Ausdruck).

## Monitoring

- [ ] **KISS-MON-01** – **Ein Uptime-Check mit Push-Benachrichtigung** (z. B. Uptime Kuma + ntfy), der dir sagt, wenn ein Dienst oder ein Host weg ist.
  → [Rezept: Monitoring](../rezepte/monitoring.md)

## Inventar

- [ ] **KISS-INV-01** – Eine **einfache Liste**: Welche Geräte, welche Dienste, welche IPs, wo ist was installiert, wie komme ich im Notfall rein. Eine Markdown-Datei oder ein Eintrag im Passwortmanager reicht.

---

**Fertig?** Glückwunsch – du hast die großen Brocken erledigt. Weiter mit [🔵 Low](01-low.md), wenn dein Setup wächst.
