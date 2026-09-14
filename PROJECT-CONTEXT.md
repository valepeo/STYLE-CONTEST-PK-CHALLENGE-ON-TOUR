# PROJECT CONTEXT — Style Contest PK Challenge On Tour

> Riassunto di contesto per riprendere il progetto in una nuova sessione/account Claude Code.
> Incolla questo file come primo messaggio (o tienilo nel repo come riferimento).

## Cos'è il progetto

App per gestire un **contest di parkour a Sacile (PN)** in stile **Pokémon GO**: gli atleti girano la città, trovano gli spot fisici, sbloccano le challenge via GPS, guardano il video dimostrativo e segnano il completamento. Classifiche live.

**Utente/organizzatore:** valepeo (peokrap@gmail.com) — organizza contest PK a squadre di 3, divisioni per taglia.

## Repository

- **GitHub:** `https://github.com/valepeo/STYLE-CONTEST-PK-CHALLENGE-ON-TOUR` (era privato, poi reso pubblico)
- **Branch di lavoro e default:** `claude/competition-management-app-gqgXI`
- **App live (anteprima):** `https://raw.githack.com/valepeo/style-contest-pk-challenge-on-tour/claude/competition-management-app-gqgXI/city-challenge.html`

## File nel repo

| File | Cosa contiene |
|---|---|
| `city-challenge.html` (~148 KB) | **L'app principale** — single file, no framework, mobile-first |
| `index.html` (~48 KB) | Prima versione: gestionale classifiche (admin competitor, leaderboard a squadre per divisione, classifica individuale). Superata ma tenuta come riferimento |

## Architettura di city-challenge.html

- **Single file HTML/CSS/JS**, tema dark navy, nessun framework
- **Leaflet.js** (CDN) + tile CartoDB Dark Matter per la mappa
- **Firebase Realtime Database v8 compat** (CDN) per sync live multi-dispositivo; **fallback completo su localStorage** ("Demo mode") se non configurato
- GPS via `watchPosition` + distanza Haversine; spot sbloccabile entro il raggio (default 50-60 m)
- Config Firebase incollabile in Admin → genera **join URL** (`#cfg=BASE64`) da condividere con gli atleti

## Modello dati (Firebase o localStorage)

```
/spots/{id}        { name, emoji, description, lat, lng, radius, needsPosition?,
                     challenges: [{id, name, points, video, hint}] }   ← array flessibile, N challenge per spot
/athletes/{id}     { name, team, division (PICCOLI|MEDI|GRANDI), joinedAt }
/completions/{id}  { athleteId, spotId, challenge (=challengeId), timestamp }
/reactions/...     reazioni 🔥💪👏 sul feed
/videoLibrary      video importati da Drive
/config/scoring    modalità punteggio; /config/organizerPinHash  PIN organizzatore
```

Migrazioni automatiche in memoria dai formati vecchi (SKILL/STYLE/SPEED) — non rompere.

## Punteggi

- **Modalità 'fixed' (default):** ogni challenge vale i suoi punti (1pt/2pt/4pt, parsati dal nome del video)
- **Modalità 'split':** punti divisi per il numero di atleti che l'hanno completata
- **Crew Bonus:** +2 a entrambi se un compagno di squadra completa la stessa challenge entro 45 min
- **XP/livelli:** 50 XP a challenge; Rookie → Explorer → Traceur → Flow Master → Street Legend
- 6 achievement derivati dai dati (First Send, Scout, Spot Sweeper, Crew Session, Hat-trick, City Runner)

## Funzionalità chiave

- **Tab:** 🗺️ Map · 🏅 Rankings (individuale + squadre, filtro divisione) · 👤 Me (profilo, XP, stats, achievement) · ⚙️ Admin (nascosto)
- **Organizer mode:** tab Admin sbloccato con PIN 4-6 cifre (da Me → "Organizer access"); hash del PIN in `/config/organizerPinHash`; il join URL NON dà accesso admin
- **Video delle challenge:** pulsante "🎥 Watch demo" → overlay con player; `toEmbedUrl()` converte link Drive (`/file/d/ID/view` → `/preview`), YouTube, mp4 diretti
- **Import ricorsivo da Google Drive** (Admin → Video Library): incolli il link della cartella → l'app la legge client-side via proxy CORS (allorigins, codetabs, corsproxy…), entra nelle sottocartelle, parsa i nomi file → anteprima → crea spot + challenge + video. Fallback: incolla link video uno per riga
- Distanza giornaliera percorsa, warm-up nudge, hint di movimento per challenge, "respect the spot"

## La cartella Google Drive dell'evento

`https://drive.google.com/drive/folders/1ZsqilZo-tVwJ_Q0E2EFIW8lY1Ht-GkyW` — "VIDEO CHALLENGES", condivisa "chiunque col link":

- Una sottocartella per spot: `1- Centro giovani zanca`, `2- Parco tallon`, `3-Rampa rotonda dietro ospedale`, `4-Comune-retro` (+ `OLD` da ignorare, uno shortcut, `Spiegazioni challenges.xlsx`)
- In ogni sottocartella: un doc "maps link" (posizione dello spot) + video challenge nominati `<ordine>-<nome> <punti>pt` (es. `1-Step cancello 1pt`, `10-Plio to cat 4pt`)
- **Gli spot reali di Sacile con coordinate dai maps link sono GIÀ caricati nell'app** (commit `d637d28`, fatto da una sessione parallela con accesso Drive)

## Stato al 2026-09-14

- Tutto committato e pushato; ultimo commit: `2bf9242` "Rework challenge flow: status selector, 10-split scoring, seed refresh"
- L'utente ha testato l'app dal telefono via raw.githack in Demo mode
- **Da fare / possibili prossimi passi:**
  - Configurare Firebase per l'evento reale (wizard già nell'app, Admin tab) e distribuire il join URL agli atleti
  - Verificare sul posto le coordinate degli spot (chip "📍 set position" se mancanti)
  - Eventuale hosting più pulito (GitHub Pages ora che il repo è pubblico)
  - Futuro desiderato dall'utente: "free map" dove gli utenti creano i propri spot

## Note operative per Claude

- Sviluppa sul branch `claude/competition-management-app-gqgXI`, commit + push sempre
- Mantieni: single-file, no framework, fallback localStorage per ogni feature, tema dark navy esistente
- L'utente lavora **da telefono**, non è tecnico: istruzioni semplici, passo-passo, in italiano
- Se serve leggere Drive: abilitare il connettore Google Drive alla creazione della sessione (a sessione avviata non si può più aggiungere)
