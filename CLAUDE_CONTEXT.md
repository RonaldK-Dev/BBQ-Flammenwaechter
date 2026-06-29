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
      burners: [{ label: string, grams: number }]   // pro Brenner kumulierte Gramm über alle Stufenwechsel
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
   - Timer läuft über Zeitstempel-Differenz (nicht setInterval-Zählung) → hintergrund-tauglich, läuft korrekt weiter wenn iOS die App in den Hintergrund legt
   - 6 Brenner einzeln, je mit Stufe: Aus/Voll/Mittel/Klein (Faktor 0/1/0.5/0.25)
   - Tippen schaltet durch die Stufen (nextLevel)
   - Echte Gramm pro Brenner werden über alle Stufenwechsel kumuliert (nicht nur Endzustand) → Verlauf zeigt z.B. "Brenner 1: 180 g"
   - Schnell-Presets: Vorheizen(alle) / 1 Brenner / Links+Rechts / Mitte(2+3)
   - Live-Verbrauch g/h, zieht beim Speichern geschätztes Gas vom currentWeight ab
   - Abbrechen-/Start-Button: transparenter "glasiger" Look
   - Tacho-Labels "0" / "100%" sauber unter den Bogenenden positioniert
5. Session-Verlauf pro Flasche mit Statistik (Anzahl, Gesamt-Gas, Gesamt-Zeit)
6. Daten-Export/-Import: lokales JSON-Backup, Import als Merge nach id (kein Datenverlust bei PWA-Löschen/Gerätewechsel)

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
- Stufen-Faktoren (0.5 / 0.25) sind Annahmen; manche Grills regeln "Klein" eher auf ~15%.
- Brenner-Verbrauchswerte (DEFAULT_BURNERS) sind BTU→g/h-Schätzwerte, noch nicht kalibriert.
  1. Test-Session (2026-06-26) zeigte ~264 g/h/Hauptbrenner als grob plausibel — der
  alte Timer-Bug (setInterval statt Zeitstempel) war der eigentliche Messfehler, nicht
  die Brennerwerte. Nächste Session mit korrektem Tracking liefert exakte Kalibrierdaten.

## Mögliche nächste Schritte
- Verbrauchswerte nach echten Test-Sessions feinkalibrieren (vorher/nachher wiegen + Backup)
- Optional später: echte iOS-App via Capacitor (localStorage → bleibt nutzbar,
  Code fast unverändert übernehmbar)
- Optional: Brenner-Werte im UI editierbar machen statt im Code

## Quelldatei
Der ursprüngliche React-Quellcode liegt auch als reine JSX vor (gas-tracker.jsx),
identische Logik, nur ohne HTML-Hülle. Bei Edits: index.html ist die maßgebliche
Datei fürs Deployment.
