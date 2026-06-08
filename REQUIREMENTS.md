# YGO Proxy Generator — Anforderungskatalog

## Projektziel

Lokal ausführbare App zum Erstellen druckfertiger Yu-Gi-Oh! Karten-Proxies für den privaten Gebrauch (z.B. Testdecks im Freundeskreis).

**Auslieferung:** Einzelne `.html`-Datei, Doppelklick im Browser öffnet die App. Kein Installer, keine Laufzeitumgebung, keine Adminrechte nötig. Weitergabe per E-Mail oder USB-Stick.

---

## Infrastruktur

**Gewählt: Einzelne HTML-Datei** (HTML + CSS + JS inline gebündelt)

| Option | Größe | Installation | Bewertung |
|---|---|---|---|
| Electron | ~150 MB | Ja | Zu groß |
| Tauri | 5–15 MB | Ja (Rust) | Gute Option für v2 |
| **HTML-Datei** | **~50 KB** | **Nein** | **Empfohlen** |

Da die Datei direkt im System-Browser läuft (nicht in einer Chat-Sandbox), funktionieren direkte API-Calls ohne Umwege.

**Externe API:** YGOPRODeck (kostenlos, keine Auth)
- Suche: `https://db.ygoprodeck.com/api/v7/cardinfo.php?fname={name}&num=8`
- Nach ID: `https://db.ygoprodeck.com/api/v7/cardinfo.php?id={id1},{id2},...`
- Kartenbilder: `https://images.ygoprodeck.com/images/cards/{id}.jpg`
- Thumbnails: `https://images.ygoprodeck.com/images/cards_small/{id}.jpg`
- Rückseite: `https://images.ygoprodeck.com/images/backs/back.jpg`

---

## Funktionale Anforderungen

### Kartensuche
- Freitextsuche nach Kartenname via YGOPRODeck API (`fname`-Parameter)
- Live-Suche mit Debounce (~400 ms)
- Bis zu 8 Treffer pro Anfrage
- Jeder Treffer zeigt: Thumbnail, Kartenname, Kartentyp
- Fehlermeldung bei fehlgeschlagener API-Anfrage

### Deck-Verwaltung
- Karte per Klick zum Deck hinzufügen
- Anzahl pro Karte einstellbar: 1–3 (TCG-Limit)
- Karte einzeln entfernen
- Gesamtanzahl der Karten wird live angezeigt

### YDK Import / Export
- **Export:** Deck als `.ydk`-Datei speichern (`#main` mit IDs, Mehrfachkopien als Wiederholungszeilen, `#extra` und `#side` leer)
- **Import:** `.ydk`-Datei laden, `#main`-Block parsen, IDs per API auflösen, Deck befüllen
- Mehrfachkopien aus YDK automatisch erkennen (max. 3)
- Kompatibel mit: EDOPro, YGOPRODeck, Dueling Nexus, Omega
- Fehlermeldung bei ungültiger/leerer YDK-Datei

### Druckvorschau & Layout
- Overlay-Ansicht mit generierten Druckseiten
- **Layout:** 3×3 Karten pro A4-Seite (9 Karten/Seite)
- **Kartenformat:** 63 × 88 mm (offizielles TCG-Format)
- Schnittmarken (Crop Marks) an allen vier Ecken jeder Karte
- Offizielle Kartenbilder von YGOPRODeck

### Rückseiten-Option
- Toggle: Rückseite ein/aus (Standard: ein)
- Bei aktiviert: nach jeder Vorderseiten-Seite folgt eine Rückseiten-Seite
- Rückseiten horizontal gespiegelt (korrekt für Duplex, lange Kante)
- Rückseiten-Bild: offizielles YGO-Kartenrücken-Design

### Druckauslösung
- Button ruft `window.print()` auf
- Hinweistext: Ränder = Keine, Hintergrundgrafiken = aktiv, Ziel = Als PDF speichern
- `@media print`: nur Druckseiten sichtbar, keine UI-Elemente

---

## Nicht-funktionale Anforderungen

- Einzelne `.html`-Datei, < 200 KB
- Keine externen Abhängigkeiten zur Laufzeit (außer Kartenbilder via CDN)
- Läuft auf Windows 10/11, macOS 12+, Ubuntu 22.04+ mit aktuellem Chrome / Firefox / Edge / Safari
- Responsiv: Zwei-Spalten-Layout auf Desktop, einspaltig auf schmalen Fenstern
- Dark Mode mit Gold-Akzenten (TCG-Stil)
- Sprache umschaltbar: Deutsch / Englisch (alle Texte in beiden Sprachen)
- Kein Tracking, keine Analytics, keine externen Scripts

---

## Out of Scope (v1.0)

- Extra Deck / Side Deck
- Manuelle Kartenerstellung (Felder selbst befüllen)
- Persistentes Speichern im Browser (kein localStorage)
- Benutzerverwaltung / Cloud-Sync
- Automatische App-Updates

---

## Roadmap (optional, v2+)

| Prio | Feature | Hinweis |
|---|---|---|
| P1 | Extra Deck & Side Deck | YDK-Sektionen `#extra` / `#side` parsen |
| P1 | Tauri-Wrapper (.exe / .app) | HTML unverändert, native Fenster-App |
| P2 | Druckformat wählbar (3×3 / 4×4) | Konfigurierbares Grid |
| P2 | Karten-Hover-Vorschau | Mouseover zeigt große Kartenansicht |
| P3 | Offline-Bildcache (IndexedDB) | Einmal laden, dann offline verfügbar |
