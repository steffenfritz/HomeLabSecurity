# SSH härten

**Level:** 🟢 KISS → 🔴 High · **Zeit:** 15 min · **Checkliste:** KISS-SYS-02, LOW-SYS-02, HIGH-SYS-04

## Warum?

SSH mit Passwort ist das Lieblingsziel von Brute-Force-Bots. Mit Schlüsseln ist Raten praktisch unmöglich.

> ⚠️ **Lass während der gesamten Änderung eine SSH-Sitzung offen** und teste in einer zweiten. Sonst sperrst du dich aus.

## Schritte

### 1. Schlüssel erzeugen (auf deinem Rechner, nicht auf dem Server)

```bash
ssh-keygen -t ed25519 -C "laptop-2026"
```

Eine **Passphrase** setzen – der `ssh-agent` bzw. der Schlüsselbund merkt sie sich, du tippst sie also selten.

### 2. Schlüssel auf den Server kopieren

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub benutzer@server
```

Jetzt testen: `ssh benutzer@server` muss **ohne Server-Passwort** funktionieren.

### 3. Server-Konfiguration

Als Drop-in-Datei, damit Paket-Updates die Einstellungen nicht überschreiben:

```bash
sudo nano /etc/ssh/sshd_config.d/10-hardening.conf
```

```text
# Nur Schlüssel
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
AuthenticationMethods publickey

# Kein Root, nur bestimmte Benutzer
PermitRootLogin no
AllowUsers benutzer

# Angriffsfläche reduzieren
MaxAuthTries 3
LoginGraceTime 30
X11Forwarding no
AllowAgentForwarding no
```

> 💡 Bei OpenSSH gilt der **erste** gefundene Wert. Dateien in `sshd_config.d/` werden alphabetisch eingelesen. `10-…` gewinnt deshalb gegen z. B. `50-cloud-init.conf`, das bei Cloud-Images gern `PasswordAuthentication yes` setzt.

### 4. Testen und neu laden

```bash
sudo sshd -t                     # Syntax prüfen – keine Ausgabe = OK
sudo sshd -T | grep -Ei 'passwordauthentication|permitrootlogin|allowusers'
sudo systemctl reload ssh        # Debian/Ubuntu; RHEL/Fedora: sshd
```

## Prüfen

- [ ] Neue Sitzung mit Schlüssel funktioniert.
- [ ] Passwort-Login wird abgelehnt:
  ```bash
  ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password benutzer@server
  # erwartet: Permission denied (publickey).
  ```
- [ ] Root-Login wird abgelehnt.

## Optional

- **Port ändern?** Reduziert Log-Rauschen, ist aber keine Sicherheitsmaßnahme. KISS: SSH gar nicht ins Internet stellen ([Mesh-VPN](ports-schliessen-mesh-vpn.md)), dann ist der Port egal.
- **Nur über VPN/LAN erreichbar:** per [Host-Firewall](host-firewall.md) oder `ListenAddress` auf die LAN-/Tailscale-IP.
- **Fail2ban/CrowdSec**, falls SSH doch exponiert ist → [Rezept](fail2ban-crowdsec.md).

## High: FIDO2-Hardware-Keys

Mit einem YubiKey, Nitrokey o. ä. liegt der private Schlüssel auf der Hardware; ohne physisches Antippen kein Login – auch Malware auf deinem Laptop kann den Schlüssel nicht stehlen.

```bash
ssh-keygen -t ed25519-sk -O resident -O verify-required -C "yubikey-1"
ssh-copy-id -i ~/.ssh/id_ed25519_sk.pub benutzer@server
```

- `resident`: Schlüssel-Handle liegt auf dem Key, auf neuem Rechner mit `ssh-keygen -K` wiederherstellbar.
- `verify-required`: zusätzlich PIN des Keys erforderlich.
- **Immer einen zweiten Key** als Ersatz registrieren.

Für viele Hosts: **SSH-Zertifikate** (OpenSSH-CA oder step-ca) statt verteilter `authorized_keys`.

## Fallstricke

- Cloud-Init-, Proxmox- oder NAS-Images haben oft eigene Drop-ins oder Weboberflächen, die SSH-Einstellungen verwalten – nach Updates prüfen.
- `AllowUsers` vergessen, Benutzer anzupassen → niemand kommt rein. Darum: zweite Sitzung offen lassen.
- Bei Ubuntu ab 22.10 kann SSH per **Socket Activation** (`ssh.socket`) laufen. Dann wirken `Port`/`ListenAddress` aus der `sshd_config` erst nach `sudo systemctl daemon-reload && sudo systemctl restart ssh.socket`.
