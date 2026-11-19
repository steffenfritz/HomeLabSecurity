# Passwörter, 2FA & Secrets

**Level:** 🟢 KISS → 🟠 Medium · **Zeit:** 1–2 h · **Checkliste:** KISS-IAM-01/02, LOW-IAM-01…04, LOW-CON-04, MED-IAM-02/03

## Warum?

Wiederverwendete und Standardpasswörter sind nach offenen Ports das größte Risiko. Und: Wer dein E-Mail-Konto übernimmt, kann fast überall das Passwort zurücksetzen.

## Schritte

### 1. Passwortmanager wählen

| Option | Für wen | Hinweis |
|---|---|---|
| **Bitwarden** (Cloud) | KISS – Familie, alle Geräte | Kein eigener Server nötig |
| **Vaultwarden** (selbst gehostet) | Low/Medium | Bitwarden-kompatibel. Wenn er ausfällt, haben Clients einen Offline-Cache – trotzdem Notfallexport einplanen |
| **KeePassXC** (Datei) | Puristen | Datei per Syncthing/Nextcloud synchronisieren; Backup nicht vergessen |

> ⚠️ **Henne-Ei-Problem:** Läuft der Passwortmanager im Homelab, brauchst du die Zugangsdaten zum Homelab *ohne* Homelab. Halte Master-Passwort, Backup-Schlüssel und Notfallzugänge auch **offline** (Ausdruck, verschlossener Umschlag) → [Notfallplan](notfallplan.md).

### 2. Standardpasswörter ersetzen

Router, NAS, Drucker, IPMI/BMC, Kameras, Smart-Home-Bridges, Switches, Access Points, Proxmox, Datenbank-Container (`POSTGRES_PASSWORD=postgres` …). Jeweils zufällig generieren lassen (≥ 20 Zeichen) und im Passwortmanager speichern.

### 3. 2FA nach Priorität

1. **E-Mail** – Schlüssel zu allem anderen
2. **Passwortmanager**
3. **Mesh-VPN-/IdP-Login** (Google/GitHub/Microsoft-Konto, mit dem du dich bei Tailscale anmeldest!)
4. **Domain-Registrar & DNS-Anbieter** – wer deine DNS-Einträge ändert, kann Zertifikate für deine Domains ausstellen
5. **Cloud-Backup / Offsite-Speicher**
6. Alles andere

Reihenfolge der Methoden, von gut nach besser: SMS < TOTP-App < Passkey / FIDO2-Hardware-Key.

**Recovery-Codes** ausdrucken und sicher verwahren.

### 4. Secrets in Docker-Setups

**KISS / Low:** Secrets in `.env`, nie in `docker-compose.yml`, nie im Git.

```bash
chmod 600 .env
echo ".env" >> .gitignore
```

```yaml
services:
  db:
    image: postgres:16.6
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
```

Passwort erzeugen: `openssl rand -base64 32`

**Besser:** Compose-Secrets – landen als Datei unter `/run/secrets/`, nicht als Umgebungsvariable (die man per `docker inspect` sieht):

```yaml
services:
  db:
    image: postgres:16.6
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

Viele offizielle Images unterstützen die `*_FILE`-Variante.

**Medium:** Secrets verschlüsselt im Git mit **SOPS + age**:

```bash
age-keygen -o ~/.config/sops/age/keys.txt
# öffentlichen Schlüssel in .sops.yaml eintragen:
cat > .sops.yaml <<'EOF'
creation_rules:
  - path_regex: .*\.enc\.env$
    age: age1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
EOF

sops --encrypt .env > app.enc.env       # committen
sops --decrypt app.enc.env > .env       # beim Deploy
```

Den privaten age-Schlüssel sichern (Passwortmanager + offline)!

## Prüfen

- [ ] Kein Passwort ist doppelt (die meisten Passwortmanager haben einen „Sicherheitsbericht“).
- [ ] `git log -p | grep -iE 'password|secret|token'` findet nichts Echtes. Gründlicher: `gitleaks detect`.
- [ ] 2FA für die Top-5-Konten aktiv, Recovery-Codes offline vorhanden.

## Fallstricke

- **Geleakte Secrets im Git** sind auch nach dem Löschen in der History. Rotieren, nicht nur löschen.
- **Umgebungsvariablen** sind für jeden mit Docker-Zugriff per `docker inspect` lesbar.
- **TOTP im selben Passwortmanager** wie das Passwort ist bequem, aber dann ist der Passwortmanager ein Single Point of Failure. Für die Top-Konten 2FA getrennt halten.
