# Feature-Prompt – Achievements / Meilensteine

Gib diesen Text an Claude Code, im Repo-Ordner geöffnet.

---

## Auftrag

Erstelle `achievements.html` als neue, dashboard-weite Sektion. Anders als
die bisherigen Seiten hat sie **keine eigene Dateneingabe** – sie liest
ausschließlich aus den bereits vorhandenen `localStorage`-Daten der anderen
Sektionen (Gym, Learning, Self-Improvement, Reflection, Intake, Finance) und
berechnet daraus freigeschaltete/nicht freigeschaltete Meilensteine.
Kategorie-Leitfarbe: `var(--gold)` (Trophäen-Charakter passt zur bereits
etablierten Finance-/Erfolgs-Farbe), einzelne Achievement-Badges behalten
aber die Farbe ihrer Quelle (siehe Tabelle unten).

## Datenmodell

Zwei Quellen: **automatisch berechnete** Achievements (siehe unten, rein
abgeleiteter Zustand aus anderen Trackern) und **manuell angelegte**
Achievements für Dinge, die kein Tracker erfasst (z. B. "Ersten Halbmarathon
gelaufen"). Beide erscheinen in derselben Ansicht, manuelle sind an einem
kleinen Tag ("manuell") erkennbar.

```js
// Automatisch berechnet – nur der Freischalt-Zeitpunkt wird persistiert
{
  achievementId: string,       // fester Slug, z.B. "gym_100kg_bench"
  unlockedAt: string | null,     // wird beim ersten Erreichen einmalig
                                    // gesetzt, danach nicht mehr verändert
  manual: false
}

// Manuell angelegt – komplett vom Nutzer definiert
{
  id: string,
  manual: true,
  icon: string,             // Emoji
  title: string,
  color: string,              // frei wählbar aus der Palette
  unlockedAt: string          // Datum, das der Nutzer selbst einträgt
}
```

"+ Eigenes Achievement"-Button öffnet ein kleines Formular: Icon (Emoji),
Titel, Datum, Farbe (Auswahl aus den bereits genutzten Token-Farben). Kein
Fortschritts-/Bedingungs-Konzept nötig für manuelle Einträge – die sind
beim Anlegen direkt freigeschaltet.

## Beispiel-Achievements (Startset, erweiterbar)

| ID | Titel | Quelle | Bedingung | Farbe |
|---|---|---|---|---|
| `streak_30` | 🔥 30-Tage-Streak | Self-Improvement | irgendein Habit 30 Tage am Stück | `var(--rose)` |
| `gym_100kg_bench` | 🏋️ 100kg Bankdrücken | Gym | Bankdrücken-Satz mit ≥100kg geloggt | `var(--orange)` |
| `linkedin_500` | 📇 500 LinkedIn-Connections | Self-Improvement (Ziel) | Ziel "LinkedIn" `current` ≥ `target` | `var(--rose)` |
| `learning_50h` | 📚 50h Learning | Learning | Summe aller Learning-Minuten ≥ 3000 | `var(--violet)` |
| `networth_10k` | 💰 10.000€ Net Worth | Finance | Net Worth ≥ 10000 | `var(--gold)` |
| `reflection_12` | 📆 12 Wochenrückblicke | Reflection | 12 Wochen-Einträge insgesamt | `var(--teal)` |
| `water_30` | 💧 30 Tage Wasserziel | Intake | 30 Tage mit erreichtem Wasserziel (nicht
zwingend am Stück) | `var(--blue)` |
| `boxing_10` | 🥊 10 Box-Sessions | Gym | 10 Boxen-Sessions insgesamt | `var(--red)` |

Die Tabelle ist ein Startset, keine feste Liste – Struktur so anlegen, dass
neue Achievements leicht als weitere Zeile in einer zentralen
Konfigurationsliste (z. B. `ACHIEVEMENTS` Array in JS) ergänzt werden
können, ohne die Berechnung anzufassen. Sobald `uni.html`/Google Kalender
existieren, können hier leicht weitere Achievements ergänzt werden.

## UI – bewusst motivierend, kein nüchternes Häkchen-System

**0. Hero-Banner (ganz oben, größtes Element der Seite):** Zeigt das
zuletzt freigeschaltete Achievement gefeiert statt nur gelistet – großes
Icon, große Überschrift ("🎉 Neuer Meilenstein!"), der Titel des
Achievements prominent, dezente Glanz-/Funkel-Deko im Hintergrund (kleine
Stern-Symbole niedriger Deckkraft, radialer Glow in der Achievement-Farbe).
Bei erstmaligem Freischalten eines neuen Achievements (nicht bei jedem
Seitenaufruf) zusätzlich eine kurze Konfetti-Animation auslösen (CSS-
Keyframes oder eine leichte JS-Canvas-Lösung, z. B. `canvas-confetti` via
CDN – kein Overengineering nötig, ein einmaliger 1–2 Sekunden Burst reicht).

**1. Rang/Level (Kopfbereich, neben dem Fortschritt):** Gesamtzahl
freigeschalteter Achievements mappt auf einen kleinen, ebenfalls
motivierenden Titel statt nur einer nackten Zahl, z. B.:

| Freigeschaltet | Rang |
|---|---|
| 0–2 | 🌱 Anfänger |
| 3–5 | 🔥 Grinder |
| 6–8 | ⚡ Champion |
| 9+ | 🏆 Legende |

(Schwellenwerte anpassen, sobald mehr Achievements als das Startset
existieren – Verhältnis grob beibehalten: unterste Stufe leicht, oberste
Stufe spürbar aufwendig.)

**2. "Neu freigeschaltet":** wie zuvor, mit Glow-Rahmen in der jeweiligen
Farbe, zusätzlich ein kleiner "NEU"-Tag oben auf der Karte.

**3. Freigeschaltete Achievements:** Grid aus Badge-Karten, volle Farbe,
Freischalt-Datum sichtbar. Manuelle Achievements bekommen einen kleinen
"manuell"-Tag in der Ecke.

**4. Noch nicht freigeschaltete Achievements:** gedimmt/entsättigt, mit
Fortschrittsbalken zum aktuellen Stand. **Wichtig für die Motivation:**
Achievements mit ≥75% Fortschritt bekommen einen hervorgehobenen "Fast
geschafft!"-Tag statt einfach nur grau in der Masse zu verschwinden – das
gibt einen klaren nächsten Ansporn.

**5. Gesamtfortschritt (Kopfbereich):** "X von Y Achievements
freigeschaltet" bleibt, aber als große, selbstbewusste Zahl statt kleiner
Randnotiz – das ist der Stolz-Moment der Seite.

## Design-Konsistenz

Gleiche Tokens, Zilla-Slab-Titel, Card-Optik mit Schatten
(`box-shadow: 0 14px 34px rgba(0,0,0,0.35)`), Section-Labels mit
Trennlinie – wie in allen bisher umgesetzten Seiten. Referenz-Mockup:
`achievements-preview.html`.

## Integration

1. `index.html`: neue Kachel "Achievements" (`--c: var(--gold)`),
   verlinkt auf `achievements.html`
2. `topbar.js`: bei Bedarf ergänzen
3. `lock.js` wie bei allen anderen Seiten einbinden
4. Kein Eintrag in `reflection.html` nötig – Achievements sind selbst schon
   eine Zusammenfassung, keine weitere Datenquelle für eine andere Seite

## Qualitätscheck

- Einmal freigeschaltete Achievements bleiben freigeschaltet, auch wenn der
  zugrunde liegende Wert später wieder sinkt (z. B. Net Worth fällt wieder
  unter 10.000€ – Achievement bleibt trotzdem freigeschaltet)
- Berechnung darf die Seite nicht spürbar verlangsamen (Daten werden beim
  Laden einmal aggregiert, nicht bei jedem Re-Render neu durchsucht)
- Mobile/Tablet/Desktop getestet
