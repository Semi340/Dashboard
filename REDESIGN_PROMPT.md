# Redesign-Prompt – Wall Street Vivid

Gib diesen Text komplett an Claude Code (in VS Code, im Repo-Ordner geöffnet).
Claude Code hat dann direkten Zugriff auf alle echten Dateien und kann sie
sicher anpassen, ohne dass Funktionalität (Sync, localStorage, iframes) verloren geht.

---

## Auftrag

Wende das folgende Design-System auf **alle** `.html`-Dateien in diesem Repo an
(index.html, health.html, gym.html, po-water.html, finance.html, caffeine.html,
nova-lite.html, main.html, template.html – und jede weitere `.html`-Datei, die du
im Repo findest). Ändere **nur** CSS-Variablen, Farben, Fonts und die unten
beschriebenen visuellen Elemente. Rühre **nichts** an der bestehenden
JavaScript-Logik, den Supabase-Sync-Aufrufen (`sync.js`, `initCloudSync`), der
`lock.js`-Einbindung, den `localStorage`-Keys oder der Datenstruktur an – nur
Optik.

## 1. Design-Tokens (in jeder Datei im `:root` ersetzen/ergänzen)

```css
:root {
  --text-primary: #F2EFE6;
  --text-secondary: #ABA79A;
  --text-tertiary: #6F6C61;
  --gold: #C9A227;
  --gold-dim: #7A611A;
  --blue: #4FA3E0;
  --red: #E0524A;
  --orange: #E0785C;
  --mint: #6FE3A0;
  --brown: #C9A36B;
  --violet: #9C8CE0;
  --font-display: "Zilla Slab", Georgia, "Times New Roman", serif;
  --font: -apple-system, BlinkMacSystemFont, "Inter", "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  --font-mono: ui-monospace, "SF Mono", Menlo, Consolas, monospace;
}
```

Behalte vorhandene `--success`/`--warning`/`--danger`-Variablen bei, falls Code
darauf zugreift, aber setze ihre Werte auf `--mint`, `--gold` bzw. `--red`.

## 2. Google Font einbinden

In jedem `<head>`, vor dem bestehenden `<style>`-Block, ergänzen:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Zilla+Slab:wght@600;700&display=swap" rel="stylesheet">
```

## 3. Hintergrund

Body-Hintergrund auf `#0A0C0B` setzen. Bestehenden `body::before`
Glow-Effekt ersetzen durch eine dezente Nadelstreifen-Textur:

```css
body::before {
  content: '';
  position: fixed; inset: 0;
  background:
    repeating-linear-gradient(100deg, rgba(255,255,255,0.012) 0px, rgba(255,255,255,0.012) 1px, transparent 1px, transparent 42px),
    radial-gradient(circle at 15% 90%, rgba(201,162,39,0.08), transparent 55%);
  pointer-events: none; z-index: -1;
}
```

Den bestehenden `body::after` (Film-Grain-Punktmuster) unverändert lassen.

## 4. Titel-Typografie

`.dash-title` und alle Section-/Page-Titel bekommen `font-family:
var(--font-display); font-weight: 700; letter-spacing: -0.01em;` statt des
bisherigen weißen Gradient-Textes. Farbe: `var(--text-primary)`.

## 5. Kategorie-Farben pro Seite

Jede Seite bekommt EINE Leitfarbe (als `--c` bzw. für Akzente wie Buttons,
aktive Zustände, Fortschrittsbalken, wichtige Zahlen):

| Seite | Kategorie-Farbe |
|---|---|
| index.html (Fitness-Kachel) | `var(--orange)` |
| health.html | `var(--mint)` |
| po-water.html | `var(--blue)` |
| finance.html | `var(--gold)` |
| gym.html | `var(--orange)` |
| caffeine.html | `var(--brown)` |
| nova-lite.html | `var(--violet)` |

## 6. Kacheln auf index.html

Jede `.tile` bekommt:
- `border-top: 2px solid var(--c)` (Kategorie-Farbe von oben)
- Einen sehr leichten radialen Farbschimmer in der Ecke:
  `radial-gradient(circle at 85% -10%, color-mix(in srgb, var(--c) 22%, transparent), transparent 60%)`
- `.tile-num` in der Kategorie-Farbe, leicht transparent

Nutze dafür pro Kachel eine eigene Klasse (`.tile-fitness`, `.tile-health`,
`.tile-water`, `.tile-finance`, `.tile-caf`, `.tile-uni`, `.tile-learn`), die
jeweils nur `--c` neu setzt – siehe Referenzcode unten.

## 7. Goal Ticker – Börsenticker-Optik

Der bestehende Goal Ticker (NASDAQ-Style, zyklisch alle 5s) bleibt in seiner
JS-Logik unverändert, bekommt aber die visuelle Hülle eines scrollenden
Tickerbands:

```css
.tape-wrap {
  background: #07080A;
  border-bottom: 1px solid rgba(201,162,39,0.25);
  overflow: hidden;
  padding: 8px 0;
  white-space: nowrap;
}
.tape {
  display: inline-block;
  font-family: var(--font-mono);
  font-size: 12px;
  letter-spacing: 0.02em;
}
.tape .up { color: var(--mint); }
.tape .down { color: var(--red); }
```

Positive/erledigte Ziele bekommen die Klasse `up`, überfällige/dringende
Ziele die Klasse `down`.

## 8. Referenz: vollständiges Mockup

Ein fertiges, lauffähiges Beispiel (nur zur optischen Referenz, nicht
1:1 übernehmen, da es keine echte Datenlogik hat) liegt bei –  orientiere
dich am Look, nicht am Inhalt:

`design-preview-wallstreet-vivid.html`

## 9. Reihenfolge

1. `topbar.js` – gemeinsame Design-Tokens/Fonts zentral pflegen, falls dort
   Styles injiziert werden
2. `index.html` – Titel, Ticker, Kacheln
3. Alle Unterseiten (health, gym, po-water, finance, caffeine, nova-lite) –
   gleiche Tokens + jeweilige Kategorie-Farbe übernehmen
4. `template.html` – als Vorlage für zukünftige neue Seiten ebenfalls
   aktualisieren

## 10. Qualitätscheck danach

- Alle Seiten auf Mobile (< 480px), Tablet (~700–800px) und Desktop testen
- `lock.js`, `sync.js`, Supabase-Verbindung müssen unverändert funktionieren
- Kontrast: Text auf farbigen Flächen immer `var(--text-primary)`, nie
  `var(--text-tertiary)`, damit es lesbar bleibt
