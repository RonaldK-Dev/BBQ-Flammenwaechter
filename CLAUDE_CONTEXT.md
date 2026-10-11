# Flammenwächter – Projekt-Kontext für Claude Code

## Was ist das Projekt
PWA-App "Flammenwächter" – Teil des BBQ-Lab Projekts. Trackt Gasflaschen-Füllstand
per Gewicht und schätzt den Grillverbrauch pro Session. Abgestimmt auf den Grill
Napoleon Rogue PRO-S 525 RSIB (4 Hauptbrenner + Heckbrenner + Sizzle Zone).

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
  (zweiter Key `gas-bottle-tracker-v1-grill-draft` für den Session-Entwurf, siehe unten)
- Sprache: Deutsch
- Kostengünstig/DIY bevorzugt (GitHub Pages statt Netlify wg. Credits)

## Dateien im Repo
- index.html              → komplette App (React-Code im <script type="text/babel">)
- manifest.json           → start_url & scope = /BBQ-Flammenwaechter/
- sw.js                   → Service Worker, Cache-Name "flammenwaechter-v5" (bei jedem Release hochzählen)
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
  ],
  refills: [             // optional, ein Eintrag pro Flaschentausch (seit 2026-10-11)
    {
      date: ISO-string,
      weight: number,              // kg, Gewicht der neuen Flasche
      previousWeight: number,      // kg, Stand der alten Flasche beim Tausch
      previousEmptyWeight: number, // kg, Tara der alten Flasche
      previousLastWeighed: ISO-string
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
   - **Absturzsicher (seit 2026-07-11):** Laufende Session wird bei jeder Brenner-
     Aktion + zusätzlich beim Verstecken der Seite (`visibilitychange`/`pagehide`)
     + alle 20s als Fallback in einem eigenen localStorage-Key
     (`gas-bottle-tracker-v1-grill-draft`) gesichert. Killt iOS die PWA (Bildschirm
     aus/Hintergrund) oder trifft ein Fingertipp aus Versehen den Modal-Backdrop,
     wird beim nächsten Öffnen von "Grillen" für dieselbe Flasche der Entwurf
     erkannt und kann per Bestätigungsdialog fortgesetzt werden (Zeit, Brennerstufen
     und kumulierte Gramm exakt wiederhergestellt, zeitstempel-basiert). Der
     Entwurf wird beim Speichern und beim bestätigten Abbrechen gelöscht.
   - **Abbrechen-Sicherheitsabfrage:** sobald schon Zeit/Gas erfasst wurde
     (elapsed > 0), fragt der Abbrechen-Button per `window.confirm` nach, bevor
     die Session verworfen wird — verhindert Datenverlust durch Ghost-Touch.
5. Session-Verlauf pro Flasche mit Statistik (Anzahl, Gesamt-Gas, Gesamt-Zeit)
   - Karten zeigen Datum + Uhrzeit (mehrere Sessions/Tag bleiben unterscheidbar)
   - Swipe-links auf einer Session-Karte (SessionRow) enthüllt "🗑 Löschen" → mit
     Bestätigung; gelöschte Session bucht ihr geschätztes Gas wieder auf currentWeight
     zurück (Umkehrung von handleGrillFinish, gedeckelt auf fullWeight)
6. Daten-Export/-Import: lokales JSON-Backup, Import als Merge nach id (kein Datenverlust bei PWA-Löschen/Gerätewechsel)
7. **Reichweiten-Prognose (seit 2026-10-11):**
   - Auf jeder Flaschen-Karte: „~X Sessions · ~Y h Grillzeit" plus die Basiswerte darunter
   - Basis (`getForecastBasis`): die letzten 10 Sessions ALLER Flaschen zusammen (der
     Verbrauch hängt am Grill, nicht an der Flasche), Sessions unter 60 s zählen nicht.
     g/h = Summe Verbrauch / Summe Dauer, Ø pro Session = Mittelwert
   - Sessions werden abgerundet (lieber eine zu wenig versprechen), ohne Sessions und bei
     leerer Flasche wird der Block ausgeblendet
   - Im Grillen-Modal zusätzlich live: „Reicht bei dieser Einstellung" = Restgas / aktueller
     Verbrauch der eingestellten Brenner
   - Die Prognose ist nur so genau wie die Brenner-Werte unten (teils unkalibriert)
8. **Lösch-Bestätigung für Flaschen (seit 2026-10-11):** 🗑️ fragt per `window.confirm` nach
   und nennt die Zahl der Sessions, die mit verloren gehen. Ein Session-Entwurf der
   gelöschten Flasche wird mit entfernt.
9. **Flaschentausch (seit 2026-10-11):** Knopf „🔄 Flasche getauscht" auf der Karte öffnet
   das SwapModal (Gewicht der neuen Flasche, vorbelegt mit Vollgewicht, und Tara).
   - Der Eintrag bleibt derselbe, alle Sessions bleiben erhalten, `refills` bekommt einen Eintrag
   - Geänderte Tara verschiebt `fullWeight` mit (Gasmenge bleibt gleich)
   - Füllungen (`fillingIndex`): Sessions werden über ihr Datum der Füllung zugeordnet.
     Karte zählt die Sessions der aktuellen Füllung plus „insgesamt N×", der Verlauf zeigt
     pro Füllung eine Überschrift (neueste zuerst). Ohne Tausch sieht alles aus wie vorher
   - Session löschen schreibt das Gas nur gut, wenn sie zur aktuellen Füllung gehört
   - Rückgängig: im Verlauf erscheint „↩️ Tausch rückgängig machen", solange in der neuen
     Füllung noch keine Session liegt (Fehl-Tipp). Stellt Gewicht, Tara und Wiege-Datum der
     alten Flasche aus dem `refills`-Eintrag wieder her und entfernt ihn. Nach der ersten
     Session der neuen Füllung gibt es den Knopf nicht mehr

## Brenner-Verbrauchswerte (DEFAULT_BURNERS in index.html)
Basis: BTU → kW (/3412) → g/h (×75 g/kWh), Vollgas:
- b1 "Brenner 1 (li)"  264 g/h  (BTU-Sollwert, noch unkalibriert)
- b2 "Brenner 2"       264 g/h  (BTU-Sollwert, noch unkalibriert)
- b3 "Brenner 3"       264 g/h  (BTU-Sollwert, noch unkalibriert)
- b4 "Brenner 4 (re)"  264 g/h  (BTU-Sollwert, noch unkalibriert)
- rear "Heckbrenner"   290 g/h  (am 2026-07-05 kalibriert, siehe unten — war 319 g/h)
- sizzle "Sizzle Zone" 308 g/h  (BTU-Sollwert, noch unkalibriert)

## Bekannte Punkte / offen
- Service Worker wirft in der LOKALEN Datei-Vorschau auf iOS den Fehler
  "Job rejected for non app-bound domain" → harmlos, nur lokal. Auf GitHub Pages
  (HTTPS) funktioniert er normal.
- Stufen-Faktoren (0.5 / 0.25) sind Annahmen; manche Grills regeln "Klein" eher auf ~15%.
- Brenner-Verbrauchswerte (DEFAULT_BURNERS) sind größtenteils BTU→g/h-Schätzwerte, noch
  nicht einzeln kalibriert (außer Heckbrenner, siehe Kalibrierung unten).
  1. Test-Session (2026-06-26) zeigte ~264 g/h/Hauptbrenner als grob plausibel — der
  alte Timer-Bug (setInterval statt Zeitstempel) war der eigentliche Messfehler, nicht
  die Brennerwerte. Nächste Session mit korrektem Tracking liefert exakte Kalibrierdaten.

## Kalibrierung Heckbrenner (2026-07-05)
Session am 04.07. (~2,5h Heckbrenner, gemischt Mittel/Voll) wurde durch einen versehentlichen
Backup-Restore in der App-Historie zerrissen (nur 37 Min. digital erfasst). Die echte Wiegung
davor/danach ist trotzdem verlässlich: 6,90 kg (29.06.) → 6,35 kg (04.07.) = **550 g realer
Verbrauch** über die komplette Session. Bei angenommenem ~50/50-Split Mittel/Voll über die 2,5h
ergibt sich ein Vollgas-Wert von ~293 g/h (gerundet 290), statt der bisherigen 319 g/h (reiner
BTU-Sollwert). Unsicherheit: exakter Mittel/Voll-Zeit-Split unbekannt, plus unklare Zusatzzeit
durch den Neustart nach dem Backup-Unfall — für exaktere Zahlen braucht es einen sauberen
Einzeltest (1 Brenner, 1 Stufe, exakte Zeit, vorher/nachher gewogen, keine Backup-Störung).

## Mögliche nächste Schritte
- Verbrauchswerte nach echten Test-Sessions feinkalibrieren (vorher/nachher wiegen + Backup)
- Optional später: echte iOS-App via Capacitor (localStorage → bleibt nutzbar,
  Code fast unverändert übernehmbar)
- Optional: Brenner-Werte im UI editierbar machen statt im Code

## Quelldatei
Der ursprüngliche React-Quellcode liegt auch als reine JSX vor (gas-tracker.jsx),
identische Logik, nur ohne HTML-Hülle. Bei Edits: index.html ist die maßgebliche
Datei fürs Deployment.
