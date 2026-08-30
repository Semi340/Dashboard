# Feature-Prompt – Boxen + Gym zusammenführen

Gib diesen Text an Claude Code, im Repo-Ordner geöffnet.

---

## Auftrag

Erweitere die bestehende `gym.html` (Progressive-Overload-Tracker) um einen
zweiten Workout-Typ **Boxen**, statt einer komplett neuen Seite. Die
bisherige Kraft-Logik (Übungen, Sätze, Wiederholungen, Gewicht,
Progressions-Tracking/PRs) bleibt vollständig erhalten und unverändert
nutzbar – Boxen kommt als zusätzlicher, strukturell anderer Eintragstyp
dazu. Kategorie-Leitfarbe bleibt `var(--orange)` (bereits als
Fitness-Farbe etabliert); Boxen-Einträge bekommen zur Unterscheidung
innerhalb der Seite `var(--red)` als Tag-Farbe (Kraft bleibt
`var(--orange)`).

## Datenmodell

Eine gemeinsame Session-Liste mit `type`-Unterscheidung:

```js
// Kraft-Session (bestehende Struktur, unverändert)
{
  id: string,
  type: "strength",
  date: string,
  exercise: string,           // z.B. "Bankdrücken"
  sets: [{ reps: number, weight: number }],
  createdAt: string
}

// Boxen-Session (neu)
{
  id: string,
  type: "boxing",
  date: string,
  rounds: number,              // Anzahl Runden
  durationMinutes: number,      // Gesamtdauer
  notes: string,                  // Technik/Fokus, z.B. "Kombinationen, Sparring"
  createdAt: string
}
```

## UI

**1. Eintragstyp-Umschalter oben im Erfassen-Formular:** 🏋️ Kraft / 🥊 Boxen
– je nach Auswahl wechseln die darunter angezeigten Felder komplett (kein
gemeinsames Formular mit optionalen Feldern, sondern zwei klar getrennte
Eingabemasken):
- **Kraft**: unverändert wie bisher (Übung, Sätze/Wdh./Gewicht)
- **Boxen**: Runden (Zahlenfeld), Dauer in Minuten (Zahlenfeld), Notizfeld
  für Technik/Fokus

**2. Verlauf:** gemeinsame, chronologische Liste beider Typen, klar
unterscheidbar durch Icon + Farbe (🏋️ orange / 🥊 rot) und typ-passende
Kurzangabe (Kraft: "Bankdrücken 3×8 @ 80kg", Boxen: "6 Runden · 24 Min.")

**3. Statistik-Kopf:** Gesamt-Sessions diese Woche (beide Typen
zusammengezählt), plus getrennt: Kraft-Progression (bestehende PR-Logik,
unverändert) und Boxen-Summe (Runden/Minuten diese Woche)

**4. Progressions-Charts:** bestehende Gewichts-/PR-Charts gelten weiter nur
für Kraft-Übungen. Für Boxen reicht eine einfache Trend-Anzeige (Minuten pro
Woche über die letzten Wochen), kein PR-Konzept nötig (Boxen hat keine
vergleichbare "Gewicht steigern"-Metrik).

## Kraft-Bereich – detaillierte Anforderungen

Prüfe zuerst, ob `gym.html` das bereits genau so umsetzt. Falls ja: nichts
ändern, nur den Boxen-Typ danebenstellen. Falls einzelne Punkte fehlen,
ergänzen, ohne bestehende Trainingsdaten zu verlieren:

- **Frei anlegbare Splits**: Der Nutzer kann eigene Trainingstage/Splits
  definieren (z. B. "Push", "Pull", "Beine" – frei benennbar, keine feste
  Liste)
- **Frei zuordenbare Übungen pro Split**: Jedem Split können beliebige
  Übungen zugewiesen werden, ebenfalls frei anlegbar/benennbar
- **Split-Start**: Wählt der Nutzer einen Split für eine neue Session, werden
  automatisch alle zugehörigen Übungen aufgelistet
- **Satz-Erfassung**: Pro Übung werden Sätze mit Gewicht und Wiederholungen
  erfasst (wie bisher)
- **Automatische Gewichts-Progression (konfigurierbar pro Übung)**: Der
  Nutzer legt pro Übung eine Regel fest – Ziel-Wiederholungen (z. B. 8) und
  Steigerungsschritt (z. B. +2,5 kg). Erreicht/übertrifft der Nutzer in
  einer Session die Ziel-Wiederholungen mit dem aktuellen Gewicht, schlägt
  die App für die nächste Session automatisch das um den Steigerungsschritt
  erhöhte Gewicht vor (anzeigen, nicht zwingend automatisch übernehmen –
  der Nutzer sieht die Empfehlung beim Start der nächsten Session für diese
  Übung und kann sie übernehmen oder manuell überschreiben)

Datenmodell-Ergänzung dafür:

```js
// Split
{ id: string, name: string, exerciseIds: string[] }

// Übung (in Splits verwendet, frei anlegbar)
{
  id: string,
  name: string,
  targetReps: number,     // Ziel-Wiederholungen für Progression, z.B. 8
  increment: number         // Steigerungsschritt in kg, z.B. 2.5
}
```

Die bestehenden Kraft-Sessions (`type: "strength"`) bekommen zusätzlich ein
optionales Feld `splitId`, um sie einem Split zuordnen zu können – bereits
vorhandene Sessions ohne `splitId` bleiben gültig und werden nicht
nachträglich zugeordnet.

## Design-Konsistenz

**Wichtig: Kein Redesign von `gym.html`.** Die Seite wurde bereits im
Rahmen von `REDESIGN_PROMPT.md` auf das Wall-Street-Vivid-Design
umgestellt (Tokens, Zilla-Slab-Titel, Card-Optik mit Schatten, orange
Akzentfarbe). Übernimm exakt die dort bereits vorhandenen Komponenten-Stile
(Cards, Buttons, Stat-Kacheln, Section-Labels, Log-Zeilen) unverändert –
nichts davon neu gestalten oder umbenennen.

Nur die **neuen** Elemente kommen dazu, im gleichen bereits etablierten
Stil: der Typ-Umschalter (Kraft/Boxen) und das Boxen-Formular. `var(--red)`
als Tag-Farbe für Boxen-Einträge ist die einzige neue Farbzuordnung.
`gym-boxing-preview.html` zeigt nur, *wie diese neuen Elemente* strukturell
aussehen könnten (Umschalter, Formular-Layout, Log-Tag-Farben) – die
bereits bestehenden Teile von `gym.html` (Stat-Kacheln, PR-Charts, generelle
Seitenstruktur) bitte 1:1 aus der aktuellen Datei übernehmen statt aus dem
Mockup nachzubauen.

## Integration mit Reflection

`reflection.html` liest bereits Gym-Daten für die 🏋️-Auto-Zeile
(Sessions-Anzahl + größte Progression). Erweitere diese Zeile beim
Aufklappen jetzt so, dass Boxen-Sessions separat mit aufgeführt werden, z. B.
"Bankdrücken +5kg · Kniebeuge +2.5kg" (Kraft) gefolgt von "🥊 2× Boxen,
48 Min. gesamt" – beide Typen sind Teil derselben Quelle (`gym.html`), die
Reflection-Seite muss also nur die neue `type`-Unterscheidung beim Lesen
berücksichtigen, keine neue Datenquelle anbinden.

## Qualitätscheck

- Bestehende Kraft-Daten/PRs bleiben nach dem Update unverändert erhalten
- Beide Eintragstypen lassen sich unabhängig voneinander bearbeiten/löschen
- Mobile/Tablet/Desktop getestet
