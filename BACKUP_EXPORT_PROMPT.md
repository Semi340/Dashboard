# Feature-Prompt – Backup-Export

Gib diesen Text an Claude Code, im Repo-Ordner geöffnet.

---

## Auftrag

Erstelle `backup.html` als neue, kleine Utility-Seite (kein Lebensbereich
wie die anderen Sektionen, sondern ein Werkzeug). `lock.js` wie bei allen
anderen Seiten einbinden. Keine neue Leitfarbe nötig – neutrale Optik
(Silber/Grau-Akzent statt einer der bunten Kategorie-Farben), da es kein
Tracking-Bereich ist.

**Wichtig zur Platzierung:** `backup.html` bekommt **keine eigene Kachel**
im Grid auf `index.html` und steht auch nicht als Punkt in der
Bottom-Nav/`topbar.js`. Stattdessen: ein kleiner, unauffälliger Icon-Button
(💾, ohne Text oder mit sehr kleinem Text) direkt auf `index.html`, fest
positioniert in einer Ecke des Hauptbereichs (z. B. oben links neben oder
über dem Seitentitel, unabhängig vom Kachel-Grid darunter) – klickbar,
führt direkt zu `backup.html`. Klein genug, dass er nicht von den
eigentlichen Lebensbereichen ablenkt, aber immer sichtbar/erreichbar,
ohne erst durch ein Menü navigieren zu müssen.

## Was exportiert wird

Alle `localStorage`-Keys, die zu den Dashboard-Sektionen gehören (Health,
Intake, Gym, Learning, Finance, Reflection, Self-Improvement, Achievements,
Habits, Ziele, Profil-Einstellungen usw. – die vollständige Liste ergibt
sich aus den tatsächlich verwendeten Keys im Repo, bitte beim Umsetzen
durchsuchen statt zu raten) sowie, falls über die bestehende
`sync.js`-Anbindung einfach erreichbar, die entsprechenden Supabase-Tabellen
des Nutzers.

**Wichtig – Sicherheit:** Folgendes NIEMALS in den Export aufnehmen:
- WHOOP-Zugriffstoken (`whoop_tokens_v1` bzw. äquivalente Keys) – Access-/
  Refresh-Token sind Zugangsdaten, kein Backup-relevanter Inhalt
- Jegliche API-Keys (z. B. der in `nova-lite.html` gespeicherte KI-API-Key)
- Supabase-Auth-Session-Daten

Diese Keys beim Export explizit herausfiltern (Blocklist im Code), nicht
nur "alles außer den offensichtlichen Tracking-Daten" exportieren.

## Export-Funktion

- Großer "Backup erstellen"-Button
- Sammelt alle relevanten Keys, verpackt sie in ein JSON-Objekt mit
  Metadaten (Export-Datum, Dashboard-Version/Kommentar), triggert einen
  Browser-Download mit Dateinamen `dashboard-backup-YYYY-MM-DD.json`
- Nach erfolgreichem Export: Zeitpunkt in `localStorage` unter einem eigenen
  Key speichern (z. B. `last_backup_at`), damit die Seite weiß, wann zuletzt
  gesichert wurde

## Import-Funktion (Wiederherstellung)

- Datei-Upload-Feld (Drag & Drop + klassischer Datei-Dialog)
- Vor dem Zurückschreiben: deutliche Warnung, dass bestehende Daten
  überschrieben werden, mit Bestätigung ("Bist du sicher?")
- Nach Bestätigung: Keys aus der JSON-Datei zurück in `localStorage`
  schreiben, danach – falls möglich – einen Sync-Trigger zu Supabase
  auslösen, damit Cloud und lokaler Stand wieder übereinstimmen
- Einfache Validierung: Datei muss die erwartete Backup-Struktur haben,
  sonst verständliche Fehlermeldung statt stillem Fehlschlag

## Status-Anzeige ("Letztes Backup")

- Kompakte Karte oben auf der Seite: "Letztes Backup: vor X Tagen"
  (oder "Noch nie", falls `last_backup_at` fehlt)
- Farbliche Kennzeichnung: aktuell (< 7 Tage) neutral, wird älter (7–14
  Tage) dezent gelb, überfällig (> 14 Tage) dezent rot – als sanfter
  Reminder, kein aufdringlicher Alarm

## Integration mit index.html (optionaler, dezenter Reminder)

Falls das letzte Backup mehr als 14 Tage her ist (oder noch nie gemacht
wurde), einen kleinen, unaufdringlichen Hinweis auf `index.html` einblenden
(z. B. eine dünne Zeile unter dem Goal Ticker: "💾 Letztes Backup ist X Tage
her – jetzt sichern"), verlinkt auf `backup.html`. Kein Modal, kein
Pop-up – das Dashboard soll nicht nervig werden, nur ein dezenter Hinweis.

## Design-Konsistenz

Gleiche Tokens, Zilla-Slab-Titel, Card-Optik mit Schatten
(`box-shadow: 0 14px 34px rgba(0,0,0,0.35)`), Section-Labels mit
Trennlinie – wie in allen bisher umgesetzten Seiten, aber insgesamt
zurückhaltender/neutraler als die farbenfrohen Tracking-Seiten. Referenz-
Mockup: `backup-preview.html`.

## Qualitätscheck

- Export enthält nachweislich keine Tokens/API-Keys (manuell die
  heruntergeladene JSON-Datei prüfen)
- Import überschreibt nur nach expliziter Bestätigung
- Reminder auf `index.html` verschwindet direkt nach einem frischen Export
- Mobile/Tablet/Desktop getestet
