# Container härten

**Level:** 🔵 Low → 🟠 Medium · **Zeit:** 30 min pro Stack · **Checkliste:** LOW-CON-01…06, MED-CON-01…03, HIGH-CON-01

## Warum?

Ein Container ist **keine** starke Sicherheitsgrenze. Läuft er als root, mit allen Capabilities und Zugriff auf den Docker-Socket, ist eine Lücke in der App schnell eine Übernahme des Hosts.

## Schritte

### 1. Die großen Drei (Low)

**Kein Docker-Socket im Container.** Wer `/var/run/docker.sock` hat, hat root auf dem Host – auch mit `:ro`, denn die API selbst erlaubt das Starten privilegierter Container.

Braucht ein Tool den Socket (Traefik, Diun, Dozzle, Homepage …), einen **Socket-Proxy** davorschalten, der nur lesende API-Aufrufe durchlässt:

```yaml
services:
  socket-proxy:
    image: tecnativa/docker-socket-proxy:<version>   # Version pinnen!
    environment:
      CONTAINERS: 1      # Container auflisten erlauben
      POST: 0            # keine schreibenden Aufrufe
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks: [socket]
    restart: unless-stopped

  traefik:
    # ...
    environment:
      - DOCKER_HOST=tcp://socket-proxy:2375
    networks: [socket, web]

networks:
  socket:
    internal: true
  web:
```

**Kein `privileged: true`**, kein `network_mode: host`, kein `pid: host` – es sei denn, du weißt genau, warum (z. B. Home Assistant mit Bluetooth, Monitoring-Agenten).

**Nur vertrauenswürdige Images**, gepinnt → [Docker-Images pinnen](docker-images-pinnen.md).

### 2. Härtungs-Vorlage (Medium)

```yaml
services:
  app:
    image: ghcr.io/example/app:1.4.2
    user: "1000:1000"                  # nicht als root
    read_only: true                    # Root-Dateisystem schreibgeschützt
    tmpfs:
      - /tmp
    volumes:
      - ./data:/data                   # nur was wirklich beschreibbar sein muss
    cap_drop:
      - ALL
    # cap_add:
    #   - NET_BIND_SERVICE             # nur falls Port < 1024 nötig
    security_opt:
      - no-new-privileges:true
    mem_limit: 512m
    pids_limit: 200
    ports:
      - "127.0.0.1:8080:8080"
    networks: [frontend, backend]
    restart: unless-stopped

  db:
    image: postgres:16.6
    volumes:
      - ./db:/var/lib/postgresql/data
    networks: [backend]                # keine Ports veröffentlicht!
    security_opt:
      - no-new-privileges:true

networks:
  frontend:
  backend:
    internal: true                     # kein Internetzugang, nicht von außen erreichbar
```

**Vorgehen:** Option für Option hinzufügen und testen. Nicht jedes Image verträgt `read_only` oder `cap_drop: ALL` – die Logs (`docker compose logs`) zeigen, was fehlt (oft: ein weiteres `tmpfs` oder eine einzelne Capability wie `CHOWN`, `SETUID`, `SETGID`).

> 💡 Für Images, die als root starten müssen, um dann Rechte abzugeben (viele LinuxServer.io-Images mit `PUID`/`PGID`), `user:` weglassen und stattdessen die vorgesehenen Variablen nutzen.

### 3. Docker-Gruppe

Wer in der Gruppe `docker` ist, kann sich jederzeit root-Rechte verschaffen. Nur Admin-Benutzer hinzufügen – oder **rootless** arbeiten:

- **Rootless Docker:** `dockerd-rootless-setuptool.sh install` (siehe Docker-Doku)
- **Podman** ist standardmäßig rootless und versteht Compose-Dateien (`podman compose`).

### 4. Prüfen mit Werkzeugen

```bash
# Host- und Daemon-Konfiguration gegen CIS-Docker-Benchmark
docker run --rm --net host --pid host --userns host --cap-add audit_control \
  -v /etc:/etc:ro -v /var/lib:/var/lib:ro -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /usr/lib/systemd:/usr/lib/systemd:ro \
  --label docker_bench_security docker/docker-bench-security

# Images auf bekannte Schwachstellen
trivy image ghcr.io/example/app:1.4.2
```

Mehr in [Audits & Scans](audits-und-scans.md).

## Prüfen

- [ ] `docker ps --format '{{.Names}}' | xargs -I{} docker inspect {} --format '{{.Name}} priv={{.HostConfig.Privileged}} user={{.Config.User}}'` – keine unerwarteten `priv=true`, möglichst wenig leere `user=`.
- [ ] Kein Container außer dem Socket-Proxy mountet `docker.sock`:
  ```bash
  docker ps -q | xargs docker inspect --format '{{.Name}} {{range .Mounts}}{{.Source}} {{end}}' | grep docker.sock
  ```
- [ ] Datenbanken haben keine veröffentlichten Ports.

## Fallstricke

- `docker.sock:ro` ist **kein** Schutz – siehe oben.
- Portainer, Dockge, Komodo & Co. brauchen vollen Socket-Zugriff. Ihre Weboberfläche ist damit faktisch eine Root-Shell: nie öffentlich, immer mit starkem Passwort/SSO und 2FA.
- Zu viel Härtung auf einmal → App startet nicht, man gibt frustriert auf. Lieber schrittweise.
