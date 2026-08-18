# PRESSOL Hub, Projekt-Kontext

Stand: 2026-08-18. Gegen `index.html` geprüft (1149 Zeilen, Commit b9f09d1).

## Was ist das?
Primärer Einzel-Datei HTML-Video-Hub für PRESSOL. Deployed auf **hub.pressol.iol.eu** via GitHub Pages (Repo: `pressol-hub`).

## Verwandtes Projekt
**MVP-App** unter `videohub.pressol.iol.eu` -> `/Users/harry/projects/2602_PRESSOL_MVP/index.html`
MVP ist die ältere Basis-Version (nur lokale Videos, kein Vimeo).

**Wichtig, leicht zu übersehen:** Der Hub hostet keine Medien. Produktvideos und deren Thumbnails werden vom MVP-Host geladen:
- MP4: `https://videohub.pressol.iol.eu/frames/${id}_${lang}.mp4` (index.html:833)
- Thumbnail: `https://videohub.pressol.iol.eu/thumbs/${id}_${lang}.jpg` (index.html:730)

Wer im MVP-Repo Medien aus dem Git-Index entfernt, nimmt dem Hub die Produktvideos mit. Dort gilt: nie `git add -A`.

## Dateistruktur
```
index.html          <- die einzige App-Datei
pressol_logo.png
CNAME               <- hub.pressol.iol.eu
CLAUDE.md           <- diese Datei
.claude/launch.json <- npx serve auf Port 3456
.claude/settings.json <- autoMemoryDirectory -> ~/knowledge/auto-memory/pressol-hub
```

## Tech Stack
- Reines HTML/CSS/JS, kein Build-System
- Tailwind CSS via CDN
- Vimeo oEmbed API für Thumbnails (CORS-frei, kein Auth)
- GA4 via gtag.js
- GitHub Pages Hosting mit Custom Domain und HTTPS (Let's Encrypt)

## Features

### Intro-Animation
- Logo materialisiert mit `blur/brightness`, 3.0s Dauer
- sessionStorage Key: `ph_intro`
- Dismiss nach 3600ms, `triggerFadeUps()` 400ms danach, `overlay.remove()` nach 800ms
- `width: clamp(120px, 26vw, 320px); height: auto;` WICHTIG: kein inline height!

### Fade-up System
- `.fade-up { opacity: 0 }` plus `.fade-up.go { animation: fadeUp }`
- `triggerFadeUps()` einmalig nach Intro
- **Kritisch**: Nach `list.innerHTML = ...` in `loadCategory()` -> double-rAF -> `.go` adden

### Vimeo Integration
- `vimeoMap` mit Video-IDs pro Sprache (7 Sprachen x 7 Videos)
- Thumbnails via `Promise.allSettled()` parallel, `vimeoThumbCache` und `item.isConnected` Guard
- Embed: `autoplay=1&title=0&byline=0&portrait=0&vimeo_logo=0&dnt=1`
- Overlay: Logo-Block (klick-blockend) plus Custom Fullscreen-Button
- User hat Vimeo Pro, `vimeo_logo=0` funktioniert

### TankFIXX Videos
Auf vimeo.com/isleoflight (persönliches Konto des Users):
```js
'tankfixx': { DE: 1169126420, EN: 1169127108, CZ: 1169125761, NL: 1169129125, FR: 1169128530, ES: 1169127876, PT: 1169129842 }
```
**Offen, live kaputt (Stand 2026-08-18):** Alle 7 IDs antworten mit Referer `hub.pressol.iol.eu` mit HTTP 401, andere Vimeo-Videos mit 200. Die oEmbed-API antwortet weiter, das Thumbnail erscheint also normal und der Fehler zeigt sich erst beim Klick. Fix nur in den Vimeo-Pro-Einstellungen: pro Video Embed auf "Anywhere" oder `hub.pressol.iol.eu` whitelisten.
Prüfbefehl: `curl -s -o /dev/null -w "%{http_code}" -H "Referer: https://hub.pressol.iol.eu/" https://player.vimeo.com/video/1169126420`

### MP4-Player-UI
`setupPlayerUI()` (index.html:936), eigene Steuerleiste statt Browser-Controls:
- Play/Pause, Seek-Bar, Volume plus Mute, Fullscreen
- Auto-Hide der Leiste, `hide-cursor` im Vollbild
- webkit-Fallbacks für iOS (`webkitEnterFullscreen`, `webkitbeginfullscreen`)

### Beschreibungstexte
- `videoDesc` (index.html:412), pro Sprache und Video, Fallback auf EN in `getDesc()`
- Collapsed per Default, `expandDesc()` über den "Mehr lesen"-Button
- Reset auf collapsed bei jedem Videowechsel

### Subline-System
```js
const videoSublines = { 'from_idea_to_reality': { DE: 'Der neue PRESSOL Firmensitz', EN: 'The new PRESSOL Headquarters', ... } }
```
Display-Bar: `#display-name | #display-subline-sep | #display-subline | SKU`

### Crossfade
Titel-Elemente faden bei Videowechsel: opacity 0 -> 250ms -> updateMeta -> opacity 1. Player-Wechsel über `#fade-overlay`, 300ms vor `doSwitch()`.

### Sprachwechsel-Animation
- `lang-exit` (slide out rechts) und `lang-enter` (slide in von links)
- Shimmer-Sweep über `#nav-wrapper`
- `_langAnim` steuert in `loadCategory()` andere Stagger-Delays als beim Intro

### Mobiler Nav-Hinweis
Einmal pro Session scrollt die Kategorie-Nav kurz an und zurück, damit die Scrollbarkeit sichtbar wird. sessionStorage Key `ph_nav_peeked`, Delay 4800ms bei laufendem Intro, sonst 700ms (index.html:1095).

### GA4
- `gtag.js`, Property `G-B8MFN67XCR` (index.html:5)
- Event `video_play` mit `video_id`, `video_type` (mp4 oder vimeo), `language`
- Feuert nur bei manuellem Klick, nicht bei Autoplay (`isAuto === false`, index.html:763)

### Kategorien
7 Stück (index.html:649):
| Key | Inhalt | Quelle |
|---|---|---|
| Schmiergeräte | 12_633, 12_062, 18_072, 12_630 | MP4 |
| Tankanlagen | 45_319, 23_163, 23_176, 13_055 | MP4 |
| Werkstatt | 19_698, 07_623, 13_012, 18_082 | MP4 |
| Pneumatik | 19_768, 18_616_051, 18_516_051, 18_870 | MP4 |
| Zubehör | 19_519, 19_529, 23_296, 23_295 | MP4 |
| Praxis | messbecher, messbecher_gastro, kanisterentlueftung, adblue_fasspumpe, tankfixx | Vimeo |
| Ueberuns | manufacturing_specialists, from_idea_to_reality | Vimeo |

### 7 Sprachen
DE, EN, CZ, NL, FR, ES, PT (Reihenfolge im Picker), vollständig i18n mit EN-Fallback in `t()`.

### Code-Schutz
- Domain-Guard: nur `hub.pressol.iol.eu`, `localhost`, `127.0.0.1`
- Anti-Inspect: F12, Ctrl/Cmd+U/S, Ctrl/Cmd+Shift+I/J/C blockiert
- Kein Rechtsklick

## Kritische Bugs (gelöst)
1. **Playlist unsichtbar nach Kategorie-Wechsel**: double-rAF Fix in `loadCategory()`
2. **Logo gestaucht**: nur width clamp, kein inline height
3. **Thumbnails langsam/stale**: Promise.allSettled plus isConnected guard
4. **TankFIXX Privacy-Error**: neue Video-IDs vom isleoflight Konto, Embed-Freigabe steht aber noch aus (siehe oben)

## Workflow
- Direkt `index.html` bearbeiten, dann `git add index.html && git commit && git push`
- GitHub Pages deployed automatisch (1 bis 2 Min), kein Staging: push ist Deploy
- Rollback: `git revert` plus push, wieder 1 bis 2 Min
- Live-Test: https://hub.pressol.iol.eu
- Lokal: `npx serve -p 3456 .` (in `.claude/launch.json` hinterlegt)

## Backport-Regel
Änderungen die technisch anwendbar sind immer auch im MVP nachführen. Nicht anwendbar sind die Vimeo-Teile: das MVP kennt nur lokale MP4, keine `vimeoMap`, keine oEmbed-Thumbnails.
