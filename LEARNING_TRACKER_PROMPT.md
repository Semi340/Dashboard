# Feature-Prompt – Learning-Tracker

Gib diesen Text an Claude Code, im Repo-Ordner geöffnet. Claude Code soll eine
neue Seite `learning.html` bauen, die sich nahtlos ins bestehende Dashboard
einfügt (gleiches Design-System, gleiche Sync-/Lock-Mechanik).

---

## Auftrag

Erstelle `learning.html` als neue Sektion, basierend auf der Struktur von
`template.html` (gleicher Grundaufbau: `lock.js`-Einbindung, `topbar.js`,
Design-Tokens aus dem bereits umgesetzten Wall-Street-Vivid-Redesign,
Supabase-Sync nach dem Muster der anderen Seiten wie `caffeine.html` oder
`gym.html`). Kategorie-Leitfarbe für diese Seite: `var(--violet)`
(`#9C8CE0`), analog zur Zuordnung in `REDESIGN_PROMPT.md`.

## Zweck

Ein kombinierter Zeit- und Notiz-Tracker für **frei verwaltbare Lern-Kategorien**
(z. B. Immobilien, Trading, KI – das sind nur Beispiele, keine feste Liste).
Der Nutzer kann jederzeit neue Kategorien anlegen, umbenennen und löschen.
Jeder Eintrag hat sowohl eine Zeitangabe als auch eine kurze Notiz – kein
reines Stoppuhr-Tool und kein reines Notizbuch, sondern beides kombiniert.

## Datenmodell

Zwei Stores: Kategorien und Einträge.

```js
// Kategorien
{
  id: string,                // uuid
  name: string,               // frei vom Nutzer vergeben
  emoji: string,               // beim Anlegen frei wählbar (kleiner Emoji-Picker
                                //  oder Textfeld für ein Emoji), Default falls leer: "📌"
  color: string,               // automatisch aus der Rotationspalette vergeben
  order: number,               // Reihenfolge in der Anzeige
  createdAt: string
}

// Einträge
{
  id: string,
  categoryId: string,          // verweist auf Kategorie.id
  date: string,                 // ISO-Datum
  minutes: number,
  note: string,
  createdAt: string
}
```

**Farbvergabe für neue Kategorien:** automatisch aus einer festen
Rotationspalette (damit alles zum Wall-Street-Vivid-Design passt, keine
beliebigen Nutzerfarben):

```js
const CATEGORY_PALETTE = [
  "var(--violet)",  // #9C8CE0 – Leitfarbe der Seite, erste Kategorie
  "var(--blue)",    // #4FA3E0
  "var(--mint)",    // #6FE3A0
  "var(--gold)",    // #C9A227
  "var(--orange)",  // #E0785C
  "var(--brown)",   // #C9A36B
  "var(--red)",     // #E0524A
];
// bei mehr als 7 Kategorien: von vorne beginnen
```

Löscht der Nutzer eine Kategorie, die noch Einträge hat: Einträge bleiben
erhalten, Kategorie wird als "Archiviert" markiert statt hart gelöscht (damit
die Statistik nicht rückwirkend Lücken bekommt).

## UI-Anforderungen

**0. Kategorien-Leiste (ganz oben, immer sichtbar):**
- Horizontale Reihe aus Pill-Chips, ein Chip pro Kategorie, jeweils mit
  kleinem Farbpunkt (Kategorie-Farbe) + Name
- Letzter Chip: `+ Kategorie` – öffnet ein kleines Inline-Formular (nur
  Namensfeld nötig, Farbe wird automatisch aus `CATEGORY_PALETTE` vergeben)
- Langes Drücken / kleines Menü-Icon an jedem Chip zum Umbenennen oder
  Archivieren (siehe Datenmodell)
- Chips scrollen horizontal, falls mehr als auf einen Blick passen (kein
  Umbruch, damit die Leiste kompakt bleibt)

**1. Schnell-Erfassen-Formular:**
- Kategorie-Auswahl: Klick auf einen der Chips aus der Kategorien-Leiste
  markiert ihn als aktiv (Ring/Border in Kategorie-Farbe), keine separate
  zweite Auswahl nötig – die Kategorien-Leiste **ist** der Selector
- Dauer-Eingabe: Zahlenfeld in Minuten, plus optional Schnellwahl-Chips
  (15 / 30 / 60 / 90 Min.)
- Notizfeld: einzeiliges Textfeld, das sich bei Bedarf zu mehrzeilig
  erweitert (kein Pflichtfeld, aber sichtbar präsent, damit es genutzt wird)
- Speichern-Button in der Farbe der aktuell gewählten Kategorie

**2. Verlauf (darunter):**
- Einträge gruppiert nach Datum (neueste zuerst), jede Zeile zeigt:
  Kategorie-Farbpunkt + Name als Pill-Label, Dauer, Notiz
- Einträge müssen editierbar/löschbar sein (Konsistenz mit anderen Seiten,
  z. B. `caffeine.html`-Log)

**3. Statistik-Kopfbereich (wie `.stat-row` im Redesign):**
- Gesamtstunden diese Woche
- Gesamtstunden diesen Monat
- Kleine Balkenübersicht: Minuten pro Kategorie (diese Woche), dynamisch
  aus der aktuellen Kategorienliste erzeugt – keine feste Anzahl Balken

## Visuelle Struktur (Übersichtlichkeit)

- Jede Kategorie zeigt ihr Emoji vor dem Namen (Chip, Formular-Button,
  Verlaufs-Einträge, Balkenübersicht) – macht Kategorien auch ohne Lesen
  des Namens auf einen Blick unterscheidbar
- Abschnitte der Seite (Neuer Eintrag / Übersicht / Verlauf) bekommen ein
  kleines Section-Label in Kapitälchen/Mono-Font mit dünner Trennlinie
  daneben, damit die Seite nicht wie ein einziger langer Block wirkt
- Die drei Stat-Kacheln (Woche/Monat/Kategorien) bekommen je ein kleines
  Emoji über dem Label (⏱️ 📅 🗂️), rein dekorativ zur schnelleren
  Orientierung

## Integration ins bestehende Dashboard

1. In `index.html`: neue Kachel `Learning` hinzufügen, Klasse
   `tile-learn` (bereits in `REDESIGN_PROMPT.md` als `--c: var(--violet)`
   vorgesehen), verlinkt auf `learning.html`
2. In `topbar.js`: falls dort eine zentrale Navigationsliste gepflegt wird,
   `Learning` dort ergänzen
3. `lock.js` muss wie bei allen anderen Seiten eingebunden werden

## Design-Konsistenz

Verwende exakt die Tokens, Fonts (Zilla Slab für Titel) und Kartenoptik aus
`REDESIGN_PROMPT.md` / `design-preview-wallstreet-vivid.html` – keine neue
Stil-Sprache erfinden, nur die violette Leitfarbe für diese Seite anwenden.

## Qualitätscheck

- Funktioniert auf Mobile (< 480px), Tablet (~700–800px), Desktop
- Sync mit Supabase funktioniert wie bei den anderen Sektionen
- Keine Kollision von `SECTION_ID` oder localStorage-Keys mit bestehenden
  Seiten
