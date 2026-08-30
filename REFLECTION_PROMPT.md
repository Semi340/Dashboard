# Feature-Prompt – Reflection (Wochen-/Monatsrückblick)

Gib diesen Text an Claude Code, im Repo-Ordner geöffnet.

---

## Auftrag

Erstelle `reflection.html` als neue Sektion, basierend auf der Struktur von
`template.html` (gleicher Grundaufbau: `lock.js`, `topbar.js`, Design-Tokens
aus dem Wall-Street-Vivid-Redesign, Supabase-Sync nach dem Muster der
anderen Seiten). Kategorie-Leitfarbe für diese Seite: neue Variable
`--teal: #5AC8B0` (bisher keiner Seite zugeordnet, passt ruhiger/kontemplativer
zum Thema Rückblick als die bereits vergebenen Signalfarben).

```css
:root {
  --teal: #5AC8B0;
}
```

## Zweck

Ein strukturierter Rückblick in zwei Rhythmen – wöchentlich und monatlich –
mit festen Prompts, damit man nicht bei null anfängt, sondern konsistent
reflektiert und Fortschritt über die Zeit sichtbar wird.

## Datenmodell

```js
{
  id: string,
  type: "week" | "month",
  periodStart: string,     // ISO-Datum, Wochen- bzw. Monatsbeginn
  periodEnd: string,        // ISO-Datum, Wochen- bzw. Monatsende
  good: string,               // "Was lief gut?"
  improve: string,             // "Was kann ich verbessern?"
  score: number,                // 1–10, subjektive Gesamtbewertung des Zeitraums
  createdAt: string
}
```

Ein Eintrag pro Woche und pro Monat – beim erneuten Öffnen eines bereits
begonnenen/abgeschlossenen Zeitraums wird der bestehende Eintrag zum
Bearbeiten geladen statt ein Duplikat anzulegen.

## UI-Aufbau

**1. Tab-Umschalter oben:** 📅 Woche / 🗓️ Monat – analog zur Tab-Optik in
`intake.html` (aktiver Tab bekommt Hintergrund in `var(--teal)`, Text dunkel).

**2. Automatische Zusammenfassung (Card, direkt unter dem Zeitraum-Titel,
vor den manuellen Prompts):**

Liest automatisch aus den `localStorage`-Daten der anderen Sektionen für den
gewählten Zeitraum (Woche bzw. Monat) und zeigt kompakte, kategorie-farbige
Stat-Zeilen – der Nutzer trägt hier nichts manuell ein, das passiert beim
Öffnen der Seite:

| Quelle | Anzeige | Farbe |
|---|---|---|
| `learning.html`-Einträge | 📚 Learning: Summe Stunden im Zeitraum | `var(--violet)` |
| `gym.html`-Einträge | 🏋️ Gym: Anzahl Sessions + größte Progression (z. B. "+5kg Bankdrücken") | `var(--orange)` |
| `intake.html` (Wasser-Teil) | 💧 Wasser: % Tage mit erreichtem Tagesziel | `var(--blue)` |
| `intake.html` (Essen-Teil) | 🍽️ Ernährung: Ø kcal/Tag (nur Tage mit Einträgen) | `var(--peach)` |
| `finance.html` | 💰 Finance: Net-Worth-Veränderung im Zeitraum (± €) | `var(--gold)` |

Falls eine Quelle im Zeitraum keine Daten hat, wird die Zeile ausgeblendet
statt "0" oder "–" anzuzeigen. Sobald `career.html`/`uni.html` existieren,
werden hier passende Zeilen ergänzt (z. B. erledigte Uni-Deadlines,
Bewerbungsstatus-Änderungen) – Struktur so anlegen, dass neue Quellen leicht
als weitere Zeile ergänzt werden können.

**Klickbare Detailtiefe:** Jede Auto-Zeile ist aufklappbar (`<details>`/
`<summary>` reicht technisch aus, kein JS-Toggle nötig) und zeigt beim
Öffnen die Aufschlüsselung hinter der Summe:

- 📚 Learning aufgeklappt: Stunden pro Kategorie aus `learning.html`
  (z. B. "🏠 Immobilien 2.1h · 📈 Trading 1.5h · 🤖 KI 0.9h"), Kategorien
  und Emojis kommen dynamisch aus den dort vom Nutzer angelegten Kategorien
- 🏋️ Gym aufgeklappt: Progression pro Übung im Zeitraum (z. B.
  "Bankdrücken +5kg · Kniebeuge +2.5kg · Klimmzüge +1 Wdh.")
- 🍽️ Ernährung: zeigt direkt zwei Werte nebeneinander statt nur kcal –
  Ø kcal/Tag **und** Ø Protein/Tag (aus den Nährwert-Feldern in
  `intake.html`); beim Aufklappen zusätzlich Ø Kohlenhydrate/Zucker/Fett
- 💧 Wasser und 💰 Finance: keine sinnvolle weitere Unterteilung vorhanden,
  bleiben ohne Aufklapp-Pfeil

## Trend: Gesamt-Balken, aufklappbar in Einzelfarben

**3. Manuelle Prompts (gleiche Card oder direkt darunter):**
  - ✅ **Was lief gut diese Woche/diesen Monat?** (Textarea)
  - 🔧 **Was kannst du verbessern?** (Textarea)
  - ⭐ **Wie würdest du den Zeitraum insgesamt bewerten?** (Score 1–10,
    als 10 kleine anklickbare Punkte/Segmente statt Zahlen-Dropdown)
- Speichern-Button in `var(--teal)`

**4. Trend (unter der Card):**
- Standardansicht: **ein** gestapelter Balken pro Woche/Monat (Segmente in
  den Kategorie-Farben übereinander, Segmenthöhe proportional zum Anteil
  dieser Kategorie am jeweiligen Wochenwert) – kompakt, aber durch die
  Segmentfarben schon auf einen Blick erkennbar, welche Kategorie diese
  Woche dominiert hat
- Klick auf einen Wochen-Balken (wieder `<details>`/`<summary>` möglich)
  fächert ihn zu den einzelnen, nebeneinander stehenden Balken pro
  Kategorie auf (wie ursprünglich besprochen), plus zusätzlicher
  Granularität direkt darunter: Learning-Unterkategorien, Gym-Übungen,
  kcal **und** Protein getrennt – dieselbe Detailtiefe wie beim Aufklappen
  der Auto-Zeilen oben, nur bezogen auf diese eine vergangene Woche
- Der subjektive Score (1–10) bleibt als kleine Zahl unter jedem
  Wochen-Balken sichtbar, auch im eingeklappten Zustand
- **Wichtig für die Lesbarkeit:** Der aufgeklappte Detailbereich einer Woche
  darf NICHT innerhalb der schmalen Wochen-Spalte gerendert werden (dort ist
  zu wenig Platz für lesbaren Text). Stattdessen erscheint er als **ein
  eigener, volle Breite einnehmender Bereich unter der gesamten
  Balkenreihe** – mit größerer Schrift (mind. 14px für Werte), größeren
  ausgefächerten Balken (mind. 90px Höhe) mit Beschriftung darüber/darunter,
  und den Detail-Infos (Learning-Kategorien, Gym-Übungen, Nährwerte) als
  kleine Karten in einem 2- oder 3-Spalten-Grid statt einer engen
  Textzeile. Technisch z. B. über CSS `:checked ~`-Sibling-Selektoren mit
  versteckten Radio-Buttons pro Woche (ein gemeinsamer Detailbereich unten,
  Inhalt wechselt je nach ausgewählter Woche) oder per JS-Toggle – beides
  ist ok, Hauptsache der Detailbereich bricht aus der schmalen Spalte aus
- Falls eine Kategorie in einer Periode keine Daten hat: Segment/Balken
  einfach weglassen, nicht auf 0 setzen
- Falls weniger als 2 Datenpunkte insgesamt vorhanden: Trend-Bereich
  ausblenden statt leeres Diagramm zu zeigen

**5. Archiv (darunter):**
- Liste vergangener Einträge, neueste zuerst, initial eingeklappt (nur
  Zeitraum + Score + kleine Farbpunkte für vorhandene Kategorien sichtbar)
- Beim Aufklappen: vollständige automatische Zusammenfassung dieser Periode
  (gleiche Zeilen wie im aktuellen Zeitraum, nur die damaligen Werte) +
  die damaligen manuellen Prompt-Antworten
- Wochen- und Monats-Einträge im jeweils aktiven Tab getrennt anzeigen

## Design-Konsistenz

Gleiche Tokens, Zilla-Slab-Titel für Überschriften, Card-Optik mit Schatten
(`box-shadow: 0 14px 34px rgba(0,0,0,0.35)`), Section-Labels mit
Trennlinie, wie in `learning-tracker-preview.html` und
`intake-preview.html` etabliert. Referenz-Mockup (nur Optik) liegt bei:
`reflection-preview.html`.

## Integration

1. `index.html`: neue Kachel "Reflection", Klasse `tile-reflection`
   (`--c: var(--teal)`), verlinkt auf `reflection.html`
2. `topbar.js`: bei Bedarf in zentraler Navigation ergänzen
3. `lock.js` wie bei allen anderen Seiten einbinden

## Qualitätscheck

- Aktuelle Woche/aktueller Monat korrekt berechnet (Wochenstart nach
  gewünschter Konvention, z. B. Montag)
- Beim erneuten Bearbeiten eines Zeitraums wird der bestehende Eintrag
  aktualisiert, kein Duplikat erzeugt
- Mobile/Tablet/Desktop getestet
