# Backups nach 3-2-1

**Level:** 🟢 KISS → 🟠 Medium · **Zeit:** 1–2 h · **Checkliste:** KISS-BAK-01…03, LOW-BAK-01…05, MED-BAK-01…04

## Warum?

Backups sind deine letzte Verteidigungslinie – gegen Ransomware, defekte Platten, kaputte Updates und eigene Fehler. **RAID ist kein Backup**, Snapshots auf derselben Platte auch nicht.

## Die 3-2-1-Regel

- **3** Kopien deiner Daten (Original + 2 Backups)
- auf **2** verschiedenen Medien/Systemen
- **1** davon außer Haus

Erweitert **3-2-1-1-0**: + 1 Kopie offline/unveränderlich, 0 Fehler beim Restore-Test.

## Schritte

### 1. Was muss gesichert werden?

| Was | Wie |
|---|---|
| Docker-Volumes / Bind-Mounts (`/srv`, `/opt/stacks`) | Dateibackup |
| Datenbanken | **Dump**, dann Dateibackup des Dumps |
| `docker-compose.yml`, Konfigurationen | Git + Dateibackup |
| `/etc` der Hosts | Dateibackup |
| VMs / LXCs | Proxmox Backup Server oder `vzdump` |
| Router-, Switch-, NAS-Konfiguration | Export-Datei, manuell nach Änderungen |
| Fotos, Dokumente | Dateibackup, **unbedingt offsite** |

### 2. Datenbanken richtig sichern

Laufende Datenbankdateien zu kopieren ergibt oft ein inkonsistentes Backup. Vorher dumpen:

```bash
# PostgreSQL
docker exec -t postgres pg_dumpall -U postgres | gzip > /srv/backup/dumps/postgres.sql.gz
# MariaDB / MySQL
docker exec mariadb sh -c 'mariadb-dump --all-databases -uroot -p"$MARIADB_ROOT_PASSWORD"' | gzip > /srv/backup/dumps/mariadb.sql.gz
```

Alternative: Stack kurz stoppen, sichern, starten.

### 3. Backup mit restic

[restic](https://restic.net) ist verschlüsselt, dedupliziert und kann viele Ziele (SFTP, S3, Backblaze B2, Hetzner Storage Box, rest-server).

```bash
sudo apt install restic

# Passwort erzeugen und SICHERN (Passwortmanager + Ausdruck!)
sudo sh -c 'openssl rand -base64 32 > /root/.restic-pass && chmod 600 /root/.restic-pass'

export RESTIC_REPOSITORY=sftp:backup@nas.lan:/backups/server1
export RESTIC_PASSWORD_FILE=/root/.restic-pass

restic init
restic backup /etc /srv /opt/stacks --exclude-caches
restic snapshots
```

**Aufräumen** (Aufbewahrungsregel):

```bash
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune
restic check
```

Ohne Passwort ist ein restic-Repository **wertlos**. Kein Passwort, kein Restore.

### 4. Automatisieren und überwachen

`/usr/local/bin/backup.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
export RESTIC_REPOSITORY=sftp:backup@nas.lan:/backups/server1
export RESTIC_PASSWORD_FILE=/root/.restic-pass
HC=https://hc-ping.com/<deine-uuid>

curl -fsS -m 10 --retry 3 "$HC/start" >/dev/null || true

docker exec -t postgres pg_dumpall -U postgres | gzip > /srv/backup/dumps/postgres.sql.gz
restic backup /etc /srv /opt/stacks --exclude-caches
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune

curl -fsS -m 10 --retry 3 "$HC" >/dev/null
```

Bei einem Fehler bricht das Skript ab (`set -e`), der Erfolgs-Ping bleibt aus und **Healthchecks** meldet sich bei dir. Das ist der entscheidende Teil: Du willst wissen, wenn ein Backup *nicht* lief.

Per Cron (`sudo crontab -e`):

```text
30 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

### 5. Offsite

Zweites Repository außer Haus, z. B. Backblaze B2, Hetzner Storage Box, S3-kompatibler Speicher oder ein NAS bei Freunden/Familie (per Tailscale verbunden). Mit restic einfach ein zweites Repository anlegen oder `restic copy` nutzen.

## Ransomware-sicher: append-only

Wenn ein Angreifer deinen Server übernimmt, hat er auch die Backup-Zugangsdaten – und löscht als Erstes die Backups. Gegenmittel:

| Methode | Wie |
|---|---|
| **restic rest-server** | Mit `--append-only` starten: Clients können schreiben, aber nicht löschen. Aufräumen (`forget --prune`) läuft separat mit eigenem Zugang. |
| **Borg** | Im `authorized_keys` des Backup-Servers: `command="borg serve --append-only --restrict-to-path /backups/server1",restrict ssh-ed25519 …` |
| **S3 Object Lock** | Bucket mit Object Lock / Retention (B2, AWS, Wasabi, MinIO). Objekte sind für X Tage unlöschbar. |
| **Pull-Backup** | Der Backup-Server holt sich die Daten; der Quell-Server hat keinen Zugang zum Backup-Ziel. |
| **Proxmox Backup Server** | Eigener API-Token pro Host mit Rolle `DatastoreBackup` (darf schreiben, nicht löschen). |
| **Offline** | USB-Platte, die nur zum Backup angeschlossen ist. |

## Prüfen – der Restore-Test

**Das ist der wichtigste Schritt.** Mindestens einmal jetzt, dann vierteljährlich:

```bash
restic snapshots
restic restore latest --target /tmp/restore-test --include /srv/nextcloud/config
diff -r /srv/nextcloud/config /tmp/restore-test/srv/nextcloud/config
```

Für einen echten Test: Einen kompletten Dienst auf einer leeren VM aus dem Backup wiederherstellen – inkl. Datenbank-Dump einspielen. Notiere, wie lange es gedauert hat und was gefehlt hat.

- [ ] Datei-Restore funktioniert
- [ ] Datenbank-Restore funktioniert
- [ ] Restore ohne Zugriff auf das Homelab möglich (Passwort offline vorhanden?)

## Fallstricke

- **Backup auf dem selben NAS**, das per SMB vom Windows-PC eingebunden ist → Ransomware verschlüsselt beides.
- **Synchronisation ist kein Backup** (Nextcloud, Syncthing, OneDrive): Gelöschte/verschlüsselte Dateien werden mitsynchronisiert.
- **Zu kurze Aufbewahrung:** Merkst du einen Fehler erst nach drei Wochen, hilft eine 7-Tage-Aufbewahrung nicht.
- **Docker-Volumes vergessen:** Named Volumes liegen unter `/var/lib/docker/volumes/`. Bind-Mounts in einem festen Verzeichnis sind leichter zu sichern.
