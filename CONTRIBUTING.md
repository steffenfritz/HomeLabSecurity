# Mitmachen

Danke, dass du helfen willst! Beiträge sind willkommen: Korrekturen, neue Checklisten-Punkte, Rezepte und Tools für die Awesome-Liste.

## Grundsätze

- **KISS zuerst.** Ein neuer Punkt sollte in das *niedrigste* Level, in dem er sinnvoll ist. Wenn etwas viel Wartung bringt, gehört es eher nach oben – und sollte das ehrlich sagen.
- **Konkret statt allgemein.** „Firewall benutzen“ hilft niemandem, „`ufw default deny incoming` + Docker-Ports an `127.0.0.1` binden“ schon.
- **Prüfbar.** Jeder Checklisten-Punkt und jedes Rezept sollte sagen, *wie* man prüft, dass es funktioniert.
- **Getestet.** Befehle bitte selbst ausprobieren und die Distribution/Version im PR nennen.
- **Sprache:** Deutsch, „du“-Form, kurze Sätze.

## Checklisten-Punkte

- Format: `- [ ] **LEVEL-BEREICH-NN** – Beschreibung.` (Bereiche siehe [checkliste/README.md](checkliste/README.md))
- Neue IDs hinten anhängen, bestehende IDs **nicht** umnummerieren (andere verweisen darauf).
- Wenn es ein Rezept gibt, verlinken: `→ [Rezept: …](../rezepte/….md)`

## Rezepte

Vorlage in [rezepte/README.md](rezepte/README.md#rezept-vorlage). Pflicht sind: Level, Zeit, betroffene Checklisten-IDs, **Prüfen** und **Fallstricke**. Neues Rezept in die Tabelle in `rezepte/README.md` eintragen.

- Versionsnummern in Beispielen als Beispiel kennzeichnen (`# Beispielversion – aktuelle prüfen`).
- Keine `:latest`-Images in Beispielen – wir predigen Pinnen, also pinnen wir auch selbst.
- Ports in Beispielen an `127.0.0.1` oder eine konkrete IP binden.

## Awesome-Liste

- Format: `- [Name](Link) – Kurze Beschreibung, was es für die Homelab-Sicherheit tut. <Level-Emoji> [💶] [🏠]`
- Link möglichst auf das Quell-Repository (bei Open Source) oder die offizielle Seite.
- Nur Projekte, die **aktiv gepflegt** werden (letztes Release/Commit in den letzten ~12 Monaten). Ausnahmen mit Hinweis.
- Alphabetisch oder nach Einstiegsfreundlichkeit innerhalb der Kategorie – bestehende Reihenfolge beibehalten.
- Kommerzielle Produkte nur mit sinnvoller kostenloser Stufe und Kennzeichnung 💶.

## Ablauf

1. Fork & Branch
2. Änderung + kurze Begründung im PR
3. Links prüfen (z. B. mit [lychee](https://github.com/lycheeverse/lychee): `lychee '**/*.md'`)
