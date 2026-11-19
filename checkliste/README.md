# Checklisten

Jedes Level baut auf dem vorherigen auf. Jeder Punkt hat eine ID (z. B. `LOW-NET-02`), damit man in Issues, Notizen oder der eigenen Doku darauf verweisen kann.

| Datei | Level | Ziel |
|---|---|---|
| [00-kiss.md](00-kiss.md) | 🟢 KISS | Die wirksamsten Maßnahmen, an einem Nachmittag umsetzbar |
| [01-low.md](01-low.md) | 🔵 Low | Solide Grundhygiene für jeden Server und jedes NAS |
| [02-medium.md](02-medium.md) | 🟠 Medium | Segmentierung, zentrale Anmeldung, Monitoring |
| [03-high.md](03-high.md) | 🔴 High | Erkennung, Härtung nach Benchmarks, Reaktionsfähigkeit |

## Bereiche

Alle Checklisten sind gleich gegliedert:

| Kürzel | Bereich |
|---|---|
| `INV` | Inventar & Dokumentation |
| `ZUG` | Zugang von außen (Remote Access) |
| `NET` | Netzwerk, Router, Firewall, DNS |
| `SYS` | Betriebssystem & Hosts |
| `CON` | Container (Docker/Podman) |
| `IAM` | Konten, Passwörter, 2FA |
| `BAK` | Backup & Wiederherstellung |
| `UPD` | Updates & Patch-Management |
| `MON` | Monitoring, Logging, Erkennung |
| `PHY` | Physische Sicherheit & Strom |
| `IR` | Notfall & Incident Response |

## So benutzt du die Checklisten

- Kopiere die Datei in deine eigene Homelab-Doku (oder forke das Repo) und hake ab.
- Ein Punkt passt nicht zu deinem Setup? Streichen und kurz begründen ist völlig in Ordnung – bewusst entschieden ist besser als blind abgehakt.
- Mach dir einen wiederkehrenden Termin (z. B. vierteljährlich), um die Checkliste erneut durchzugehen.
