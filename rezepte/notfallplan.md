# Notfallplan

**Level:** 🔵 Low → 🔴 High · **Zeit:** 1 h · **Checkliste:** LOW-INV-02, LOW-IR-01, MED-IR-01/02, HIGH-IR-01…03

## Warum?

Im Notfall bist du gestresst, und genau die Systeme, die du bräuchtest (Passwortmanager, Wiki, VPN), sind vielleicht weg. Ein Plan, der **offline** liegt, macht aus Panik eine Abarbeitung.

## Teil 1: Der Notfallzettel (Low)

Ausdrucken, in einen Umschlag, an einen sicheren Ort. Optional zweites Exemplar bei einer Vertrauensperson.

```text
HOMELAB NOTFALLZETTEL – Stand: 2026-11

Netz
  Router:            http://192.168.10.1   Benutzer: ____   Passwort: → Passwortmanager "Router"
  Router-Notfall:    Reset-Knopf 15 s, Konfig-Backup liegt auf: ____
  Subnetze:          LAN 192.168.10.0/24 · Server 10.42.20.0/24 · IoT 10.42.30.0/24

Zugänge
  Passwortmanager:   Master-Passwort: ______________________ (oder: im versiegelten Umschlag B)
  2FA-Recovery-Codes:  Umschlag B
  Mesh-VPN:          Login über ____ (Konto), Admin-Konsole: ____
  Notzugang ohne VPN: lokal per Monitor/Tastatur an Server ____, Benutzer ____

Backups
  restic-Repos:      sftp:backup@nas:/backups/*   ·   b2:bucket-name:/server1
  restic-Passwort:   Umschlag B
  Restore-Anleitung: siehe Rückseite

Wiederaufbau-Reihenfolge
  1. Router/Firewall  2. Switch/WLAN  3. DNS  4. Hypervisor  5. Backup-Server
  6. Passwortmanager  7. IdP/SSO  8. Rest nach Wichtigkeit
```

## Teil 2: Was tun bei Verdacht auf Kompromittierung? (Low)

Anzeichen: unbekannte Prozesse, hohe CPU-Last ohne Grund (Miner), neue Benutzer oder SSH-Keys, verschlüsselte Dateien, Warnungen von Monitoring/CrowdSec/Wazuh, Mails vom Provider über Spam/Angriffe von deiner IP.

1. **Ruhe bewahren. Nicht sofort alles löschen.**
2. **Isolieren:** Netzwerkkabel ziehen bzw. VM-Netzwerk trennen, Port-Freigaben schließen. Strom *nicht* sofort ziehen, wenn du noch analysieren willst – im Homelab ist schnelles Isolieren aber wichtiger als Forensik.
3. **Notieren:** Was ist aufgefallen, wann? Screenshots, `ps aux`, `ss -tupan`, `last`, `journalctl --since …` sichern.
4. **Von einem sauberen Gerät** Passwörter ändern: zuerst E-Mail, Passwortmanager, Mesh-VPN, dann alle Zugänge, die auf dem betroffenen System gespeichert waren.
5. **Schlüssel rotieren:** SSH-Keys, API-Tokens, Mesh-VPN-Keys des Geräts entfernen (in der Admin-Konsole), Cloud-Zugangsdaten, Backup-Zugänge.
6. **Neu aufsetzen statt reparieren:** Ein kompromittiertes System ist nicht mehr vertrauenswürdig. Neu installieren, Daten aus einem Backup **vor** dem Vorfall zurückspielen.
7. **Ursache finden und beheben** (offener Port? veraltete App? schwaches Passwort?), bevor das System wieder ans Netz geht.
8. **Betroffene informieren**, wenn Daten anderer betroffen sein könnten (Familie, Freunde mit Konten auf deinen Diensten).

## Teil 3: Playbooks (High)

Kurze, konkrete Abläufe pro Szenario. Beispiel:

### Ransomware auf einem Client

- [ ] Client sofort vom Netz (WLAN aus, Kabel ab)
- [ ] NAS-Freigaben prüfen: Welche Dateien wurden verändert? Ggf. SMB-Dienst stoppen
- [ ] Snapshots des NAS (ZFS/Btrfs) prüfen – sind sie intakt?
- [ ] Append-only-/Offline-Backup prüfen
- [ ] Client neu aufsetzen; Daten aus Snapshot/Backup wiederherstellen
- [ ] Alle Zugangsdaten, die auf dem Client gespeichert waren, ändern

### Gerät verloren / gestohlen (Laptop, Handy)

- [ ] Gerät in Tailscale/NetBird-Admin entfernen
- [ ] Sitzungen in Passwortmanager, E-Mail, IdP beenden
- [ ] SSH-Key des Geräts aus allen `authorized_keys` entfernen
- [ ] Ggf. Fernlöschung auslösen

### Zugangsdaten geleakt (z. B. Token versehentlich auf GitHub)

- [ ] Token **sofort widerrufen** (nicht nur aus dem Repo löschen)
- [ ] Logs des Dienstes prüfen: Wurde er benutzt?
- [ ] Neues Token erzeugen, nur mit nötigen Rechten

## Teil 4: Üben (High)

Einmal im Jahr: Einen Dienst „verlieren“ (VM löschen – eine Kopie behalten!) und nur mit Notfallzettel und Backups wiederherstellen. Zeit stoppen, Lücken im Plan notieren und beheben.

## Fallstricke

- Der Notfallplan liegt **nur** im Wiki, das im Homelab läuft.
- Das Backup-Passwort liegt **nur** im Passwortmanager, der im Homelab läuft.
- Niemand außer dir weiß, dass es den Zettel gibt.
