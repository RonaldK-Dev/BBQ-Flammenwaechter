# Flammenwächter – Projekt-Kontext für Claude Code

## Was ist das Projekt
PWA-App "Flammenwächter" – Teil des BBQ-Lab Projekts. Trackt Gasflaschen-Füllstand
per Gewicht und schätzt den Grillverbrauch pro Session. Abgestimmt auf den Grill
Napoleon Rogue SE 525 RSIB (4 Hauptbrenner + Heckbrenner + Sizzle Zone).

## Tech-Stack (bewusst gewählt)
- Standalone `index.html` mit React 18 + Babel über CDN – KEIN Build-Step
  (gleiches Muster wie die bestehende BBQ-Lab Tasting-App)
- Reines `localStorage`, kein Backend, kein Account, kein Sync
- PWA: manifest.json + sw.js (Service Worker, offline-fähig)
- Deployment: GitHub Pages, eigenes Repo `BBQ-Flammenwaechter`
  (User: RonaldK-Dev → URL: https://ronaldk-dev.github.io/BBQ-Flammenwaechter/)

## WICHTIGE Dev-Präferenzen (vom User)
- Targeted Edits (str_replace) statt komplette Rewrites
- Vor größeren Änderungen warnen
- Bestehende Daten NIE verlieren
- localStorage-Key NIEMALS ändern: `gas-bottle-tracker-v1`
- Sprache: Deutsch
- Kostengünstig/DIY bevorzugt (GitHub Pages statt Netlify wg. Credits)

## Dateien im Repo
- index.html              → komplette App (React-Code im <script type="text/babel">)
- manifest.json           → start_url & scope = /BBQ-Flammenwaechter/
- sw.js                   → Service Worker, Cache-Name "flammenwaechter-v1"
- icon.svg                → Quelldatei (Flammen-Monster, BBQ-Lab Stil)
- icon-192.png / icon-512.png / icon-180.png / apple-touch-icon.png
- favicon.png / favicon-16.png
- README.md               → Deploy-Anleitung
- CLAUDE_CONTEXT.md       → diese Datei

## Datenmodell (localStorage, Key: gas-bottle-tracker-v1)
Array von bottle-Objekten:
```
{
  id: string,
  name: string,
  gasType: "Propan" | "Butan" | "Camping Gas",
  emptyWeight: number,   // kg, Leergewicht (Tara, aufgestempelt)
  fullWeight: number,    // kg, Voll = Leer + Gasmenge
  currentWeight: number, // kg, aktuell gewogen
  lastWeighed: ISO-string,
  sessions: [
    {
      id: string,
      date: ISO-string,
      durationSeconds: number,
      consumedKg: number,
      burners: string[]   // Labels der aktiven Brenner
    }
  ]
}
```

## Kern-Features (alle implementiert)
1. Mehrere Flaschen anlegen/bearbeiten/löschen
2. Füllstand-Berechnung: pct = (currentWeight - emptyWeight) / (fullWeight - emptyWeight)
   - Zeigeranzeige (SVG GaugeArc), Farbe grün>40% / orange>15% / rot
3. "Wiegen"-Modal: aktuelles Gewicht eingeben → Füllstand aktualisiert
4. "Grillen"-Modal (Session-Tracker):
   - Timer (setInterval, sekundenweise)
   - 6 Brenner einzeln, je mit Stufe: Aus/Voll/Mittel/Klein (Faktor 0/1/0.5/0.25)
   - Tippen schaltet durch die Stufen (nextLevel)
   - Schnell-Presets: Vorheizen(alle) / 1 Brenner / Links+Rechts / Mitte(2+3)
   - Live-Verbrauch g/h, zieht beim Speichern geschätztes Gas vom currentWeight ab
5. Session-Verlauf pro Flasche mit Statistik (Anzahl, Gesamt-Gas, Gesamt-Zeit)

## Brenner-Verbrauchswerte (DEFAULT_BURNERS in index.html)
Basis: BTU → kW (/3412) → g/h (×75 g/kWh), Vollgas:
- b1 "Brenner 1 (li)"  264 g/h
- b2 "Brenner 2"       264 g/h
- b3 "Brenner 3"       264 g/h
- b4 "Brenner 4 (re)"  264 g/h
- rear "Heckbrenner"   319 g/h
- sizzle "Sizzle Zone" 308 g/h
→ Das sind Schätzwerte bei Vollgas. Nach echten Wiegungen kalibrieren.

## Bekannte Punkte / offen
- Service Worker wirft in der LOKALEN Datei-Vorschau auf iOS den Fehler
  "Job rejected for non app-bound domain" → harmlos, nur lokal. Auf GitHub Pages
  (HTTPS) funktioniert er normal.
- Timer läuft via setInterval → zählt NICHT korrekt weiter wenn iOS die App in den
  Hintergrund legt. Fix für später: Verbrauch über Zeitstempel-Differenz statt
  Sekunden-Hochzählen berechnen.
- Stufen-Faktoren (0.5 / 0.25) sind Annahmen; manche Grills regeln "Klein" eher auf ~15%.

## Mögliche nächste Schritte
- Verbrauchswerte nach echten Test-Sessions kalibrieren
- Hintergrund-tauglicher Timer (Zeitstempel-Diff)
- Optional später: echte iOS-App via Capacitor (localStorage → bleibt nutzbar,
  Code fast unverändert übernehmbar)
- Optional: Brenner-Werte im UI editierbar machen statt im Code

## Quelldatei
Der ursprüngliche React-Quellcode liegt auch als reine JSX vor (gas-tracker.jsx),
identische Logik, nur ohne HTML-Hülle. Bei Edits: index.html ist die maßgebliche
Datei fürs Deployment.
