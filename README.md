# HomeLabSecurity

> In a world of constant threats, give my homelab some peace.

Eine pragmatische Sammlung, um **private Homelabs** – Server, Netzwerk, NAS, Container, IoT – abzusichern.
Ohne Enterprise-Overkill, aber auch ohne „wird schon nichts passieren“.

Das Repo besteht aus drei Teilen:

| Teil | Wofür | Einstieg |
|---|---|---|
| ✅ **Checklisten** | Was sollte ich tun? Sortiert nach Sicherheits-Level | [`checkliste/`](checkliste/README.md) |
| 🍳 **Rezepte** | Wie mache ich es konkret? Schritt für Schritt, mit Befehlen | [`rezepte/`](rezepte/README.md) |
| ⭐ **Awesome-Liste** | Welche Tools gibt es? Monitoring, SIEM, VPN, Backup, … | [`AWESOME.md`](AWESOME.md) |

---

## Die Level

Die Level bauen **aufeinander auf**. Wer *Medium* umsetzt, hat *KISS* und *Low* bereits erledigt.

| Level | Für wen? | Aufwand | Wartung | Typische Maßnahmen |
|---|---|---|---|---|
| 🟢 **KISS** | Alle. Wirklich alle. | ein Nachmittag | ~0 | Keine Portfreigaben, Mesh-VPN, Passwortmanager, Auto-Updates, Backup, Images pinnen |
| 🔵 **Low** | Ein paar Dienste, ein Server/NAS | ein Wochenende | wenige Min./Monat | Host-Firewall, SSH-Härtung, 3-2-1-Backup, Update-Benachrichtigungen, USV |
| 🟠 **Medium** | Mehrere Hosts, Familie nutzt Dienste, evtl. etwas öffentlich erreichbar | mehrere Wochenenden | ~1 h/Monat | VLANs, Reverse Proxy + SSO, zentrales Monitoring (Checkmk & Co.), Container-Härtung, Immutable Backups |
| 🔴 **High** | Sensible Daten, öffentlich erreichbare Dienste, Lern-/Lab-Zwecke für den Beruf | fortlaufend | regelmäßig | SIEM/XDR (Wazuh), IDS (Suricata), FIDO2-Keys, CIS-Härtung, Schwachstellenscans, Incident-Response |

### KISS – Level *und* Prinzip

**Keep it simple, stupid** ist hier zweierlei:

1. **Ein eigenes Level**: die ~12 Maßnahmen mit dem besten Verhältnis von Aufwand zu Wirkung. Wer nur das umsetzt, ist bereits besser geschützt als die meisten.
2. **Ein Prinzip für alle Level**: Komplexität ist selbst ein Sicherheitsrisiko. Jede zusätzliche Komponente muss gepatcht, verstanden und überwacht werden.

> **Faustregeln**
> - Was du nicht betreibst, kann nicht gehackt werden. Abschalten schlägt absichern.
> - Was nicht im Internet hängt, wird nicht vom Internet angegriffen. Mesh-VPN schlägt Portfreigabe.
> - Ein Backup, das nie zurückgespielt wurde, ist eine Hoffnung, kein Backup.
> - Ein Sicherheitstool, das du nicht wartest, ist ein weiteres Einfallstor.
> - Lieber *Low* vollständig als *High* zur Hälfte.

---

## Wo fange ich an?

1. Arbeite die [KISS-Checkliste](checkliste/00-kiss.md) ab. Das sind die wirksamsten Maßnahmen.
2. Die zwei wichtigsten Rezepte dazu:
   - [Ports schließen & Mesh-VPN (Tailscale/NetBird)](rezepte/ports-schliessen-mesh-vpn.md)
   - [Docker-Images auf Versionen pinnen](rezepte/docker-images-pinnen.md)
3. Danach das nächste Level, wenn dein Setup oder dein Risiko wächst – nicht, weil es geht.

## Bedrohungsmodell (kurz)

Wogegen schützen wir uns realistisch?

- **Automatisierte Scans & Bots** aus dem Internet, die bekannte Lücken und Standardpasswörter ausnutzen (häufigstes Risiko).
- **Ransomware** – auf dem eigenen Rechner eingefangen, verschlüsselt sie alles, was per Netzlaufwerk erreichbar ist.
- **Kompromittierte IoT-Geräte** im selben Netz wie NAS und Server.
- **Supply Chain** – manipulierte oder plötzlich kaputte Container-Images (`latest`).
- **Eigene Fehler** – versehentlich gelöschte Daten, kaputte Updates, defekte Platten.
- **Physisch** – Diebstahl, Stromausfall, Überspannung, Wasser.

Nicht im Fokus: gezielte Angriffe staatlicher Akteure. Wer davon betroffen ist, braucht mehr als ein Repo.

---

## Hinweise

- Versionsnummern in Beispielen sind **Beispiele**. Prüfe immer die aktuelle Version des jeweiligen Projekts.
- Befehle beziehen sich meist auf **Debian/Ubuntu**. Für andere Distributionen ist das jeweils vermerkt, wo es sich unterscheidet.
- Lies Install-Skripte (`curl … | sh`), bevor du sie ausführst.
- Alles ohne Gewähr. Teste Änderungen an SSH und Firewall immer mit einer zweiten, offenen Sitzung.

## Mitmachen

Ergänzungen, Korrekturen und neue Rezepte sind willkommen – siehe [CONTRIBUTING.md](CONTRIBUTING.md).

## Lizenz

[GPL-3.0](LICENSE)
