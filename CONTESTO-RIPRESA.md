Questo file serve solo per riprendere il progetto dopo il cambio account — puoi cancellarlo una volta che il lavoro riprende normalmente.

# CONTESTO RIPRESA — Style Contest PK Challenge On Tour

## Cosa stiamo facendo

App per un **contest di parkour a Sacile (PN)** in stile **Pokémon GO**: gli atleti girano la città, trovano gli spot fisici, sbloccano le challenge via GPS, guardano il video dimostrativo della challenge e segnano il completamento. Classifiche live. Organizzatore: valepeo — contest a squadre di 3, divisioni PICCOLI/MEDI/GRANDI.

## Repository

- **GitHub:** https://github.com/valepeo/STYLE-CONTEST-PK-CHALLENGE-ON-TOUR (pubblico)
- **Branch di lavoro e default:** `claude/competition-management-app-gqgXI`
- **App live:** https://raw.githack.com/valepeo/style-contest-pk-challenge-on-tour/claude/competition-management-app-gqgXI/city-challenge.html
- File: `city-challenge.html` (~148 KB, app principale) · `index.html` (~48 KB, primo gestionale classifiche, superato) · `PROJECT-CONTEXT.md` (riassunto gemello di questo)

## Stato attuale (2026-09-14)

- Tutto committato e pushato; ultimo commit: `4d758eb`
- **Gli spot reali di Sacile sono già caricati nell'app** (coordinate estratte dai doc "maps link" della cartella Drive, commit `d637d28`, fatto da una sessione parallela con accesso al connettore Drive)
- L'utente ha testato l'app dal telefono in Demo mode; Firebase per l'evento reale non ancora configurato

## Architettura dell'app (city-challenge.html)

- Single file HTML/CSS/JS, nessun framework, tema dark navy, mobile-first
- Leaflet.js (CDN) + tile CartoDB Dark Matter per la mappa; GPS con watchPosition + Haversine, sblocco entro raggio (50-60 m)
- **Firebase Realtime DB v8 compat** per il sync live; **fallback totale su localStorage** ("Demo mode") se non configurato; config incollabile in Admin → genera join URL `#cfg=BASE64` per gli atleti
- Tab: 🗺️ Map · 🏅 Rankings (individuale + squadre, filtro divisione) · 👤 Me (profilo, XP, achievement) · ⚙️ Admin (nascosto, sbloccato con PIN 4-6 cifre da Me → "Organizer access"; hash in `/config/organizerPinHash`; il join URL NON dà admin)

## Modello dati

```
/spots/{id}        { name, emoji, description, lat, lng, radius, needsPosition?,
                     challenges: [{id, name, points, video, hint}] }   ← N challenge per spot
/athletes/{id}     { name, team, division, joinedAt }
/completions/{id}  { athleteId, spotId, challenge (=challengeId), timestamp }
/reactions/…       reazioni 🔥💪👏 sul feed · /videoLibrary video importati
/config/scoring    modalità punteggio · /config/organizerPinHash PIN
```
Migrazioni automatiche in memoria dai formati vecchi (SKILL/STYLE/SPEED): non rompere.

## Decisioni prese

1. Da gestionale classico (index.html) → app GPS stile Pokémon GO (city-challenge.html)
2. Modello challenge flessibile: array per spot con punti propri (1pt/2pt/4pt parsati dai nomi dei video), niente più trio fisso SKILL/STYLE/SPEED
3. **Punteggi:** modalità 'fixed' (default: punti flat della challenge) o 'split' (punti ÷ completatori); Crew Bonus +2 se un compagno completa la stessa challenge entro 45 min; XP 50/challenge con livelli Rookie→Street Legend; 6 achievement
4. Video delle challenge riprodotti in-app da Google Drive (`toEmbedUrl()` converte link Drive/YouTube/mp4)
5. Import ricorsivo della cartella Drive lato client via proxy CORS (l'ambiente cloud di Claude NON raggiunge Drive: policy di rete) — con anteprima e fallback incolla-link
6. Admin protetto da PIN; atleti vedono solo Map/Rankings/Me

## Cartella Google Drive dell'evento

https://drive.google.com/drive/folders/1ZsqilZo-tVwJ_Q0E2EFIW8lY1Ht-GkyW — "VIDEO CHALLENGES", condivisa "chiunque col link". Una sottocartella per spot (`1- Centro giovani zanca`, `2- Parco tallon`, `3-Rampa rotonda dietro ospedale`, `4-Comune-retro`, + `OLD` da ignorare, `Spiegazioni challenges.xlsx`); dentro: doc "maps link" (posizione) + video `<ordine>-<nome> <punti>pt` (es. `1-Step cancello 1pt`, `10-Plio to cat 4pt`).

## Prossimi passi / in sospeso

- [ ] Configurare Firebase per l'evento reale (wizard 5 passi già in Admin, con "Test connection") e distribuire il join URL agli atleti
- [ ] Verificare sul posto le coordinate degli spot (chip arancione "📍 set position" dove mancano)
- [ ] Hosting più pulito ora che il repo è pubblico (es. GitHub Pages)
- [ ] Desiderio futuro dell'utente: "free map" dove gli utenti creano i propri spot
- [ ] Leggere `Spiegazioni challenges.xlsx` dal Drive (mai analizzato)

## Note operative per Claude

- Sviluppare sul branch `claude/competition-management-app-gqgXI`, commit + push sempre
- Mantenere: single-file, no framework, fallback localStorage per ogni feature, tema dark navy
- L'utente lavora **da telefono** e non è tecnico: istruzioni semplici, passo-passo, in italiano
- Per leggere Drive: abilitare il connettore Google Drive **alla creazione della sessione** (a sessione avviata non si può aggiungere; l'ambiente cloud non raggiunge drive.google.com via rete)
