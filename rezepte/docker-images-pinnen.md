# Docker-Images auf Versionen pinnen

**Level:** 🟢 KISS → 🟠 Medium · **Zeit:** 15–60 min · **Checkliste:** KISS-CON-01, LOW-CON-05, MED-CON-05, MED-CON-06, HIGH-CON-02

## Warum?

`image: nginx:latest` bedeutet: „Gib mir irgendwas.“ Beim nächsten `docker compose pull` kommt vielleicht ein Major-Update mit Breaking Changes, eine Datenbank-Migration, die nicht zurück geht – oder im schlimmsten Fall ein kompromittiertes Image.

Pinnen bedeutet: **Du entscheidest, wann sich etwas ändert.** Das macht Updates nachvollziehbar, reproduzierbar und rückgängig machbar.

## Die Stufen

| Schreibweise | Bedeutung | Bewertung |
|---|---|---|
| `nginx` / `nginx:latest` | Irgendeine Version | ❌ Nie verwenden |
| `postgres:16` | Neueste 16.x | ⚠️ KISS-Kompromiss: Bugfixes automatisch, keine Major-Sprünge |
| `nginx:1.27` | Neueste 1.27.x | ⚠️ Wie oben |
| `nginx:1.27.3` | Genau diese Version | ✅ Empfehlung |
| `nginx:1.27.3@sha256:…` | Genau dieses Image, Byte für Byte | ✅✅ Unveränderlich (Tags können überschrieben werden, Digests nicht) |

> 💡 Bei **Datenbanken** (Postgres, MariaDB, MongoDB, Redis) mindestens die **Major-Version** pinnen. Ein Major-Upgrade braucht fast immer eine manuelle Migration.

## Schritte

### 1. Bestehende `:latest` finden

```bash
# Alle Compose-Dateien durchsuchen
grep -rnE 'image:[[:space:]]*[^:@[:space:]]+(:latest)?[[:space:]]*$' --include='*compose*.y*ml' .

# Welche Version läuft gerade wirklich?
docker ps --format 'table {{.Names}}\t{{.Image}}'
docker image inspect <image> --format '{{json .Config.Labels}}' | jq
```

Viele Images haben ein Label `org.opencontainers.image.version`, das die echte Version verrät. Sonst: Release-Seite des Projekts oder die Oberfläche der App.

### 2. Version eintragen

```yaml
services:
  nginx:
    image: nginx:1.27.3          # vorher: nginx:latest
  db:
    image: postgres:16.6         # vorher: postgres
```

Dann `docker compose up -d` – es sollte sich nichts ändern, wenn du die laufende Version eingetragen hast.

### 3. (Medium) Zusätzlich mit Digest pinnen

```bash
docker pull nginx:1.27.3
docker image inspect nginx:1.27.3 --format '{{index .RepoDigests 0}}'
# nginx@sha256:0a399eb1…
```

```yaml
    image: nginx:1.27.3@sha256:0a399eb1...   # Tag ist für Menschen und Renovate, der Digest zählt
```

Ist ein Digest angegeben, wird der Tag ignoriert – du bekommst garantiert dasselbe Image.

### 4. Über neue Versionen informiert werden: Diun

Pinnen ohne Updates wäre auch schlecht. **Diun** (Docker Image Update Notifier) prüft regelmäßig und benachrichtigt dich – aktualisiert aber **nichts selbst**. Das ist der empfohlene Ersatz für das nicht mehr gepflegte Watchtower.

```yaml
services:
  diun:
    image: crazymax/diun:4.28.0        # Beispielversion – aktuelle prüfen
    command: serve
    volumes:
      - ./data:/data
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - TZ=Europe/Berlin
      - DIUN_WATCH_SCHEDULE=0 6 * * *
      - DIUN_PROVIDERS_DOCKER=true
      - DIUN_PROVIDERS_DOCKER_WATCHBYDEFAULT=true
      - DIUN_NOTIF_NTFY_ENDPOINT=https://ntfy.sh
      - DIUN_NOTIF_NTFY_TOPIC=<langes-zufälliges-topic>
    restart: unless-stopped
```

Damit Diun bei gepinnten Images auch **neuere Tags** meldet (nicht nur neue Builds desselben Tags), pro Container Labels setzen:

```yaml
    labels:
      - diun.enable=true
      - diun.watch_repo=true
      - diun.include_tags=^\d+\.\d+\.\d+$
```

Alternative mit Weboberfläche: **What's Up Docker (WUD)**.

### 5. Update-Ablauf am Patch-Tag

```bash
# 1. Release Notes lesen (Breaking Changes?)
# 2. Backup / Snapshot
# 3. Version in compose-Datei erhöhen
docker compose pull
docker compose up -d
docker compose logs -f --tail=50
# 4. Funktioniert? → committen. Kaputt? → alte Version eintragen, up -d
git commit -am "nginx 1.27.3 -> 1.27.4"
```

## Renovate

**(Medium)** Liegen deine Compose-Dateien in einem Git-Repo (GitHub, GitLab, Forgejo/Gitea), erstellt **Renovate** automatisch Pull Requests für neue Versionen – inklusive Changelog-Link. Du mergst, wenn du Zeit hast; ein Deploy-Skript oder du selbst führt `docker compose up -d` aus.

`renovate.json` im Repo:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended", "docker:pinDigests"],
  "packageRules": [
    {
      "matchDatasources": ["docker"],
      "matchUpdateTypes": ["major"],
      "labels": ["major-update"],
      "automerge": false
    }
  ]
}
```

`docker:pinDigests` ergänzt automatisch die Digests. Renovate erkennt `docker-compose.yml`, `compose.yaml` und Dockerfiles von selbst.

## Prüfen

- [ ] `grep` aus Schritt 1 findet nichts mehr.
- [ ] Ein `docker compose pull` ohne Änderung an der Datei holt keine neuen Images.
- [ ] Du bekommst Benachrichtigungen über neue Versionen.

## Fallstricke

- **Watchtower & Auto-Updates:** Watchtower wird nicht mehr gepflegt – ein ungepflegtes Tool mit Zugriff auf den Docker-Socket ist selbst ein Risiko. Und automatisches Aktualisieren auf `latest` ist das Gegenteil von Pinnen: Breaking Changes merkst du erst, wenn etwas kaputt ist. Wer Watchtower noch einsetzt, ersetzt es durch **Diun** (siehe Schritt 4) – das benachrichtigt nur, du entscheidest über das Update.
- **Pinnen ≠ sicher:** Ein gepinntes Image, das nie aktualisiert wird, sammelt Sicherheitslücken. Pinnen braucht einen Update-Prozess (Diun/Renovate + Patch-Tag).
- **Gleitende Tags** wie `stable`, `release`, `lts` oder `nightly` sind genauso unbestimmt wie `latest`.
- **Multi-Arch:** Der Digest aus `RepoDigests` ist der Digest der Manifest-Liste (gilt für alle Architekturen). Das ist in der Regel das, was du willst.

## Weiter

- [Container härten](container-haerten.md)
- [Audits & Scans](audits-und-scans.md) – Images mit Trivy auf bekannte Lücken prüfen.
