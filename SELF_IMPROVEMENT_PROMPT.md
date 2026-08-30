# Feature-Prompt – Self-Improvement

Gib diesen Text an Claude Code, im Repo-Ordner geöffnet.

---

## Auftrag

Erstelle `self-improvement.html` als neue Sektion, basierend auf der
Struktur von `template.html` (Grundaufbau, `lock.js`, `topbar.js`,
Design-Tokens des Wall-Street-Vivid-Designs, Supabase-Sync nach dem Muster
der anderen Seiten). Kategorie-Leitfarbe: neue Variable `--rose: #D98BA5`
(bisher keiner Seite zugeordnet).

```css
:root {
  --rose: #D98BA5;
}
```

**Kein Karriere-/Notizen-Bezug.** Diese Seite ist bewusst auf tägliche
Selbstverbesserungs-Gewohnheiten und persönliche Mini-Ziele fokussiert
(z. B. LinkedIn-Netzwerk, Sozialleben, Lesen, Aufräumen, Skincare, No
Snooze). Karriere-Notizen im engeren Sinn (Bewerbungen, berufliche Ziele)
gehören woanders hin (normale Notizen) und sind hier explizit nicht
Teil des Scopes.

## Zwei Item-Typen (frei anlegbar, wie beim Learning-Tracker)

**1. Habit (tägliches Abhaken)** – für Dinge wie Skincare, No Snooze,
Aufräumen, Lesen, Socials:

```js
{
  id: string,
  type: "habit",
  name: string,
  emoji: string,
  color: string,          // automatisch aus Rotationspalette
  order: number,
  createdAt: string
}

// Log-Eintrag pro Tag
{
  id: string,
  itemId: string,
  date: string,             // ISO-Datum
  done: boolean,
  note: string | null         // optionale kurze Notiz, z.B. "Küche aufgeräumt"
}
```

**2. Ziel (allgemein anpassbar – Fortschrittsbalken mit Zahl, optionalem
Datum)** – funktioniert für beliebige Ziele, nicht nur LinkedIn: Name,
Zielzahl und Einheit sind komplett frei wählbar (z. B. "500 Connections",
"12 Bücher", "10.000€ Erspartes"), `current` jederzeit direkt in der Karte
editierbar, `deadline` optional:

```js
{
  id: string,
  type: "goal",
  name: string,
  emoji: string,
  color: string,
  target: number,             // z.B. 500
  current: number,             // aktueller Stand, manuell aktualisierbar
  unit: string,                  // z.B. "Connections"
  deadline: string | null,        // ISO-Datum, optional
  order: number,
  createdAt: string
}
```

**Farbvergabe:** automatisch aus Rotationspalette, analog zum
Learning-Tracker:

```js
const SI_PALETTE = [
  "var(--rose)", "var(--gold)", "var(--mint)", "var(--blue)",
  "var(--violet)", "var(--peach)", "var(--orange)",
];
```

## UI

**1. "+ Neuer Punkt"-Button:** öffnet ein kleines Formular – Name, Emoji,
Typ-Auswahl (Habit / Ziel). Bei Ziel zusätzlich: Zielzahl, Einheit,
optionales Datum.

**2. Sektion "Heute" (Habits):** Checkliste der Habit-Items für den
aktuellen Tag, jedes Item als anklickbare Zeile mit Häkchen, Emoji, Name –
Farbe des jeweiligen Items als Häkchen-/Rahmenfarbe beim Abhaken. Direkt
darunter ein kleiner Streak-Hinweis pro Item (z. B. "No Snooze – 5 Tage
Streak"). Zusätzlich ein kleines Notiz-Icon (📝) pro Zeile: öffnet ein
einzeiliges Textfeld für eine kurze Notiz zum heutigen Eintrag (z. B. bei
"Aufräumen" – "Küche aufgeräumt"). Notiz ist optional, wird beim
Zurückblicken (Wochenübersicht, Reflection) mit angezeigt, wenn vorhanden.

**3. Sektion "Ziele":** Karten mit Fortschrittsbalken (`current`/`target`,
z. B. "LinkedIn – 120/500 Connections"), Balken in der Item-Farbe, falls
`deadline` gesetzt: Restzeit anzeigen (z. B. "noch 45 Tage"). `current`
lässt sich direkt in der Karte hochzählen/bearbeiten (kein separates
Log-Konzept wie bei Habits nötig).

**4. Wochenübersicht (Habit-Grid):** kompaktes Raster – Zeilen = Habits,
Spalten = letzte 7 Tage, Zelle gefüllt in Item-Farbe wenn an dem Tag
erledigt, sonst leer. Klassisches Habit-Tracker-Grid, sofort erkennbar wo
Lücken sind.

## Design-Konsistenz

Gleiche Tokens, Zilla-Slab-Titel, Card-Optik mit Schatten
(`box-shadow: 0 14px 34px rgba(0,0,0,0.35)`), Section-Labels mit
Trennlinie – wie in allen bisher umgesetzten Seiten. Referenz-Mockup:
`self-improvement-preview.html`.

## Integration

1. `index.html`: neue Kachel "Self-Improvement" (`--c: var(--rose)`),
   verlinkt auf `self-improvement.html`
2. `topbar.js`: bei Bedarf ergänzen
3. `lock.js` wie bei allen anderen Seiten einbinden
4. `reflection.html`: neue Auto-Zeile ergänzen, z. B. "🌱 Self-Improvement:
   5/6 Habits heute · LinkedIn 120/500" – **als eine einzige, nicht
   aufklappbare Zeile** (wie die bestehenden Wasser-/Finance-Zeilen in
   Reflection), keine Unterteilung nach einzelnen Habits nötig. Gleiche
   Datenquelle (`self-improvement.html`), keine neue Anbindung erforderlich

## Qualitätscheck

- Neue Items lassen sich anlegen, umbenennen, löschen (Logs/Fortschritt
  bleiben bei Löschung archiviert statt hart gelöscht, analog zur
  Kategorien-Logik im Learning-Tracker)
- Habit-Grid und Streak-Berechnung korrekt bei Tageswechsel
- Mobile/Tablet/Desktop getestet
