# 🔥 Flammenwächter

Gasflaschen-Tracker für BBQ-Lab. Füllstand per Gewicht berechnen, Grillverbrauch pro Session schätzen (abgestimmt auf Napoleon Rogue SE 525), mehrere Flaschen verwalten.

## Features
- Füllstand per Gewicht (Leer / Voll / Aktuell) mit Zeigeranzeige
- Mehrere Flaschen speicherbar (lokal auf dem Gerät)
- Grill-Sessions mit Timer und Verbrauchsschätzung
- 6 Brenner einzeln mit Stufen (Voll / Mittel / Klein / Aus)
- Schnell-Presets (Vorheizen, 1 Brenner, Links+Rechts, Mitte)
- Session-Verlauf mit Statistik
- Offline nutzbar (PWA)

## Daten
Alles wird lokal im Browser gespeichert (`localStorage`, Key: `gas-bottle-tracker-v1`).
Keine Cloud, kein Account. Daten bleiben auf dem jeweiligen Gerät.

## Deploy auf GitHub Pages

1. Auf **github.com** ein neues Repository erstellen:
   - Name: **BBQ-Flammenwaechter**
   - **Public**
   - **Create repository**

2. Im Repo auf **"uploading an existing file"** klicken und ALLE Dateien hochladen:
   - `index.html`
   - `manifest.json`
   - `sw.js`
   - `icon-192.png`
   - `icon-512.png`
   - `icon-180.png`
   - `apple-touch-icon.png`
   - `favicon.png`
   - `favicon-16.png`
   - `icon.svg` (optional, Quelldatei)
   - `README.md` (optional)

3. **Commit changes**

4. **Settings → Pages → Branch: main → /(root) → Save**

5. Nach ~1 Minute erreichbar unter:
   **https://ronaldk-dev.github.io/BBQ-Flammenwaechter/**

## Aufs iPhone installieren
1. Link in **Safari** öffnen
2. **Teilen-Symbol** → **"Zum Home-Bildschirm"**
3. Icon erscheint wie eine echte App, startet im Vollbild

## Verbrauchswerte anpassen
Die g/h-Werte pro Brenner stehen in `index.html` im Block `DEFAULT_BURNERS`.
Nach ein paar echten Wiegungen kannst du sie dort feinjustieren.

---
Teil des BBQ-Lab Projekts 🔥
