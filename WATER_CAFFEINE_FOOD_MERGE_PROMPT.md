# Feature-Prompt – Water + Koffein + Essen zusammenführen

Gib diesen Text an Claude Code, im Repo-Ordner geöffnet.

---

## Auftrag

Führe `po-water.html` und `caffeine.html` zu **einer** neuen Seite
`intake.html` zusammen und ergänze einen dritten Bereich für einen einfachen
Essens-Log. Die bestehende fachliche Logik aus `po-water.html` (Profil
Gewicht/Alter/Aktivität, Substanz-Datenbank die den Wasserbedarf beeinflusst,
inkl. der Koffein-Wirkung auf den Wasserbedarf) bleibt vollständig erhalten –
sie wird nur in die neue Seite integriert statt in eine separate Datei
ausgelagert.

**Wichtig:** `po-water.html` wird aktuell per iframe in `health.html`
eingebettet (siehe Prüfung `window.self !== window.top` zur
Sync-Konflikt-Vermeidung). Passe diese Einbettung so an, dass `health.html`
künftig `intake.html` einbettet – die iframe-Logik/Sync-Guard muss dabei
erhalten bleiben.

## Struktur der neuen Seite: 3 Tabs

Ein Tab-Umschalter oben, drei Bereiche, jeder mit eigener Akzentfarbe:

| Tab | Emoji | Akzentfarbe |
|---|---|---|
| Wasser | 💧 | `var(--blue)` (`#4FA3E0`) |
| Koffein | ☕ | `var(--brown)` (`#C9A36B`) |
| Essen | 🍽️ | `var(--peach)` (`#E3A17E`, neu – siehe unten) |

Neue CSS-Variable ergänzen (Palette bisher hatte noch keine passende Farbe
für "Essen"):

```css
:root {
  --peach: #E3A17E;
}
```

Der Tab-Umschalter selbst bleibt neutral (kein Akzent), aber der aktive Tab
übernimmt die Akzentfarbe seines Bereichs (Border/Text in der jeweiligen
Farbe), damit man auch am Umschalter sofort sieht, wo man ist.

## Gemeinsamer Kopfbereich (über den Tabs, immer sichtbar)

Kompakte Stat-Zeile mit dem Tageswert aus allen drei Bereichen gleichzeitig
(damit man nicht durch alle Tabs klicken muss, um den Überblick zu haben):

- 💧 Wasser heute (ml / Ziel)
- ☕ Koffein heute (mg)
- 🍽️ Kalorien heute (kcal, grobe Schätzung)

## Tab: Wasser (bestehende Logik aus po-water.html)

Unverändert übernehmen: Profil-Einstellungen (Gewicht/Alter/Aktivität),
Substanz-Datenbank (Diuretika, Stimulanzien etc.) inkl. deren Einfluss auf
den berechneten Wasserbedarf, Tages-Log der Wasseraufnahme. Nur die Optik an
`var(--blue)` als Akzentfarbe anpassen (Buttons, Fortschrittsring/-balken,
aktive Zustände).

**Verknüpfung mit dem Koffein-Tab:** Koffein-Einträge aus dem Koffein-Tab
sollen automatisch in die Wasserbedarfs-Berechnung einfließen (die
Substanz-Datenbank kennt Koffein ja bereits als wasserbedarf-beeinflussenden
Stoff) – die beiden Tabs teilen sich also die Koffein-Tagesmenge als
gemeinsamen Datenpunkt, nicht zwei getrennte Zähler.

## Tab: Koffein (bestehende Logik aus caffeine.html)

Unverändert übernehmen: Intake-Log mit Zeitstempel, Timing-Anzeige (z. B.
verbleibende Wirkzeit/Halbwertszeit-Visualisierung, falls vorhanden). Optik
auf `var(--brown)` umstellen.

## Tab: Essen (neu, einfacher Log – bewusst kein voller Makro-Tracker, aber
mit den wichtigsten groben Nährwerten)

**Datenmodell:**

```js
{
  id: string,
  date: string,        // ISO-Datum
  time: string,         // HH:MM
  name: string,          // z.B. "Haferflocken mit Banane"
  kcal: number | null,    // grobe Schätzung, optional
  protein: number | null, // Gramm, optional
  carbs: number | null,   // Kohlenhydrate in Gramm, optional
  sugar: number | null,   // davon Zucker in Gramm, optional
  fat: number | null,     // Fett in Gramm, optional
  createdAt: string
}
```

**UI:**
- Schnell-Erfassen: Name (Textfeld) + kcal (Zahlenfeld) immer sichtbar,
  darunter eine ausklappbare Zeile "Nährwerte" mit vier kompakten
  Zahlenfeldern nebeneinander: Protein (g) 🥩 · Kohlenhydrate (g) 🍚 ·
  davon Zucker (g) 🍬 · Fett (g) 🥑 – alle optional, bewusst kein Pflichtfeld,
  damit die schnelle Erfassung nicht zur Hürde wird
- Log-Liste gruppiert nach Tag: Uhrzeit, Name, kcal groß, darunter eine
  kleine graue Zeile mit den angegebenen Makros (z. B. "P 24g · K 60g ·
  davon Z 8g · F 12g") – nur die Werte anzeigen, die auch eingetragen
  wurden, keine Nullen auflisten
- Tages-Summe: kcal groß im Tab-Kopf, darunter vier kleine Makro-Chips mit
  Tagessumme (Protein/Kohlenhydrate/Zucker/Fett), im gleichen Chip-Stil wie
  die Balkenübersicht im Learning-Tracker
- Kein Lebensmittel-Datenbank-Lookup, keine Portionsgrößen-Berechnung –
  reine Freitext-Schnellerfassung mit optionalen Zahlen

## Visuelle Struktur (Übersichtlichkeit, wie beim Learning-Tracker)

- Abschnitte bekommen Section-Labels in Kapitälchen/Mono-Font mit dünner
  Trennlinie (z. B. "Heute" / "Nährwerte" / "Verlauf")
- Log-Einträge werden nach Tag gruppiert (Tag-Label in Mono-Font,
  Kapitälchen), neueste zuerst
- Jeder Log-Eintrag bekommt ein passendes Emoji vor dem Text (💧 Wasser,
  ☕ Koffein, 🍽️ Essen), auch wenn alle drei Tabs mittlerweile in einer
  gemeinsamen Liste denkbar wären – Emoji macht auch bei gemischter
  Darstellung sofort klar, worum es geht
- Cards mit leichtem Schatten (`box-shadow: 0 14px 34px rgba(0,0,0,0.35)`),
  wie in `learning-tracker-preview.html` etabliert
- Referenz-Mockup (nur Optik): `intake-preview.html`

## Integration

1. `health.html`: iframe-Einbettung von `po-water.html` auf `intake.html`
   umstellen
2. `index.html`: bestehende Kacheln/Links zu `po-water.html` und
   `caffeine.html` (falls als eigene Kacheln vorhanden) durch eine
   gemeinsame Kachel "Intake" ersetzen, verlinkt auf `intake.html`, Akzent
   `var(--blue)` als Leitfarbe der Kachel
2. `topbar.js`: Wasser-Pill in der Topbar (falls dort verlinkt) weiter auf
   `intake.html` (Wasser-Tab) zeigen lassen
3. Alte Dateien `po-water.html` und `caffeine.html` können danach entfernt
   werden – vorher sicherstellen, dass keine anderen Seiten noch direkt
   darauf verlinken

## Qualitätscheck

- Sync/localStorage-Keys sauber zusammenführen, keine Datenverluste beim
  ersten Laden nach dem Update (bestehende Wasser-/Koffein-Daten müssen
  automatisch in die neue Struktur übernommen werden, nicht auf Null
  zurückgesetzt werden)
- iframe-Einbettung in `health.html` weiterhin ohne Sync-Konflikte
- Mobile/Tablet/Desktop getestet
