# Wazuh einrichten

**Level:** 🔴 High · **Zeit:** 1 Tag · **Checkliste:** HIGH-MON-01, HIGH-UPD-01, HIGH-IR-02, HIGH-SYS-06

## Warum?

**Wazuh** ist eine Open-Source-Plattform für SIEM und XDR. Agenten auf deinen Hosts sammeln Logs und Zustände, der Server wertet aus und alarmiert. Für das Homelab interessant:

- **File Integrity Monitoring (FIM):** Wer hat `/etc/passwd`, `authorized_keys` oder `/usr/bin` verändert?
- **Security Configuration Assessment (SCA):** Prüfung gegen CIS-Benchmarks pro Host
- **Vulnerability Detection:** Welche installierten Pakete haben bekannte CVEs?
- **Log-Analyse & Erkennungsregeln:** Brute-Force, Rootkits, verdächtige Befehle, Docker-Events
- **Active Response:** z. B. IP automatisch sperren

> ⚠️ **KISS-Check:** Wazuh besteht aus Indexer (OpenSearch-basiert), Server und Dashboard und braucht spürbar Ressourcen – für eine Single-Node-Installation realistisch **4 CPU-Kerne und 8 GB RAM** aufwärts plus Plattenplatz für Logs. Wer nur Uptime und Plattenplatz sehen will, ist mit [Monitoring](monitoring.md) besser bedient.

## Voraussetzungen

- Eigene VM (empfohlen) mit Docker und Docker Compose
- Hosts mit Netzwerkzugriff auf den Wazuh-Server (Ports 1514/1515 TCP)

## Schritte

### 1. Kernel-Parameter für den Indexer

```bash
echo 'vm.max_map_count=262144' | sudo tee /etc/sysctl.d/99-wazuh.conf
sudo sysctl --system
```

### 2. Single-Node per Docker

Aktuelle Version auf der Release-Seite von `wazuh/wazuh-docker` nachschauen und als Branch/Tag verwenden:

```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.x.y   # konkrete Version einsetzen
cd wazuh-docker/single-node

# Zertifikate für die interne Kommunikation erzeugen
docker compose -f generate-indexer-certs.yml run --rm generator

docker compose up -d
```

Dashboard: `https://<wazuh-vm>:443`.

### 3. Standard-Passwörter ändern – sofort

Die Docker-Installation kommt mit **bekannten Standard-Passwörtern** (für Dashboard-Admin, interne API-Benutzer). Diese laut offizieller Anleitung *„Changing the default password of Wazuh users“* für die Docker-Installation ändern, **bevor** Agenten angebunden werden. Dashboard nur aus MGMT-VLAN oder über VPN erreichbar machen.

### 4. Agenten installieren

Im Dashboard: *Agents management → Deploy new agent* – Betriebssystem wählen, Server-Adresse und Gruppe eintragen. Das Dashboard erzeugt den passenden Befehl, z. B. für Debian/Ubuntu sinngemäß:

```bash
WAZUH_MANAGER="10.42.10.20" WAZUH_AGENT_GROUP="linux-server" \
  apt-get install wazuh-agent      # nach Einrichten des Wazuh-Repos
sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-agent
```

Agenten-Version **nicht neuer** als die Server-Version installieren.

### 5. Sinnvolle Grundkonfiguration

In der Agenten-Gruppenkonfiguration (*Agents management → Groups → linux-server → Files*, `agent.conf`):

```xml
<agent_config>
  <!-- FIM: Echtzeitüberwachung wichtiger Pfade -->
  <syscheck>
    <directories realtime="yes" check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
    <directories realtime="yes" check_all="yes">/root/.ssh,/home</directories>
    <ignore type="sregex">.log$|.swp$</ignore>
  </syscheck>

  <!-- Docker-Ereignisse -->
  <wodle name="docker-listener">
    <disabled>no</disabled>
  </wodle>
</agent_config>
```

Hinweis: `/home` in Echtzeit kann auf Desktop-Systemen viel Rauschen erzeugen – auf Server-Pfade wie `.ssh` beschränken. Der Docker-Listener benötigt auf dem Agenten Python und das `docker`-Python-Paket (siehe Wazuh-Doku *Monitoring Docker*).

### 6. Benachrichtigungen

Wazuh alarmiert standardmäßig nur im Dashboard. Für Push:

- **E-Mail** über `<global><email_notification>` im Manager (`ossec.conf`), mit Mindest-Level (z. B. `<email_alert_level>10</email_alert_level>`)
- **Integrationen** (Slack, eigene Webhooks → z. B. ntfy) über `<integration>`-Blöcke

Mit hohem Alarmlevel starten (≥ 10) und langsam senken.

### 7. Active Response (vorsichtig)

Beispiel: IP nach SSH-Brute-Force für 10 Minuten sperren – im Manager `ossec.conf`:

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5763</rules_id>
  <timeout>600</timeout>
</active-response>
```

Vorher testen, eigene Netze in die `white_list` des Managers aufnehmen.

## Prüfen

- [ ] Alle Agenten im Dashboard *Active*.
- [ ] Test-FIM: `sudo touch /etc/wazuh-test` → Alarm im Dashboard *File Integrity Monitoring*.
- [ ] Test-Login: mehrfach falsches SSH-Passwort/Key → Alarm *sshd: authentication failed*.
- [ ] *Configuration Assessment* zeigt pro Host einen CIS-Score – die wichtigsten fehlgeschlagenen Checks durchgehen.
- [ ] *Vulnerability Detection* zeigt Ergebnisse – mit Patch-Tag abgleichen.

## Fallstricke

- **Alarmflut:** Am Anfang viele Meldungen. Regeln mit eigenen Regeln (Level 0) gezielt ausblenden, nicht das ganze Feature abschalten.
- **Plattenplatz:** Der Indexer wächst. Index-Lifecycle (ISM-Policy) für die Aufbewahrung einrichten, z. B. 90 Tage.
- **Upgrades:** Manager, Indexer und Dashboard müssen versionsgleich aktualisiert werden. Upgrade-Anleitung pro Version lesen, vorher Snapshot der VM.
- **Wazuh selbst ist ein lohnendes Ziel:** Der Manager kann über Active Response Befehle auf allen Agenten ausführen. Zugang streng absichern.

## Weiter

- **Suricata** auf OPNsense für Netzwerk-Erkennung, Logs in Wazuh einspielen
- **Security Onion** – komplette Distribution für Netzwerk-Sicherheitsmonitoring (noch ressourcenhungriger)
