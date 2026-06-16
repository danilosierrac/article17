# ARTICLE 17 — Project Context

> *You submitted a GDPR deletion request to Lethe Corp.*
> *ERASURE-7 has been dispatched to your location.*

Working context for continuing development on this game. For player-facing info see [README.md](README.md).

## Concept

A browser-based top-down horror/stealth game. You play **Mara Voss**, a data subject whose Article 17 (GDPR "right to erasure") request has been assigned to **ERASURE-7** — a corporate enforcement unit that physically resolves open deletion tickets by eliminating the data subject.

The horror is bureaucratic, not supernatural. ERASURE-7 doesn't hate you — it has an open ticket, a 72-hour SLA, and it will close both. **Lethe Corporation** ("Your data, handled with care", 100% deletion compliance) is the antagonist institution.

Visual/tonal references: [NOCT (2015)](https://store.steampowered.com/app/394460/NOCT/) for thermal/night-vision top-down perspective; SA-X chase sequences from Metroid Fusion for a relentless, non-defeatable pursuer.

## Win/Lose conditions

- Collect 5 evidence files scattered through the Lethe Corp. building
- Press **F** to file a Legal Hold once all 5 are collected — freezes the deletion queue, ERASURE-7 cannot process a contested record
- **Win**: Legal Hold filed before ERASURE-7 reaches you → "LEGAL HOLD ACTIVE"
- **Lose**: ERASURE-7 catches you → "REQUEST RESOLVED / SUBJECT: CLOSED"

## Tech stack

Pure HTML5 Canvas, single file (`index.html`), no dependencies, no build step. Deployed via GitHub Pages.

- Repo: `danilosierrac/article17` on GitHub
- Live: https://danilosierrac.github.io/article17
- Local preview: `npx serve -l 3417 .` (configured in `.claude/launch.json` as `article17`)

## Implementation history (chronological)

1. **v1 initial commit** — basic playable prototype (movement, hide, evidence pickup, legal hold)
2. **v2 NOCT overhaul** — darkness/FOV system, ERASURE-7 directional scan cone, speed rebalance (enemy faster than player), noise radius rings on movement, tightened hide mechanic
3. **Win-screen freeze fixes** (two rounds):
   - Bug: `endGame()` had `if(state!=='playing')return` — pressing F set `state='ending'`, which made `endGame()` bail before showing the win screen
   - Fix: condition changed to `if(state!=='playing'&&state!=='ending')return`
   - Bug 2: canvas froze for the ~2.6s `'ending'` delay because the game loop itself stopped on state change
   - Fix: loop keeps running during `'ending'`; only `update()` and the timer increment are gated to `'playing'`
4. **Boot sequence + NES-style procedural music** — Lethe Corp. terminal boot screen on load, Web Audio API music with 3 intensity states (patrol/alert/chase) driven by ERASURE-7's `aware` value, mute toggle (M key + HUD button)
5. **Horror audio overhaul** (current):
   - Replaced NES-action-style arpeggios with sparse, dissonant patterns built on the D→Ab tritone ("devil's interval")
   - Patrol: very slow (spb=0.48), long silences, single notes
   - Alert: chromatic half-step grinding (D→Eb→E→F)
   - Chase: fast tritone stabs alternating D↔Ab
   - Added a persistent D1 (36.7Hz) sawtooth sub-bass drone running under all three states
   - Die chime: sub-bass thud + noise burst + slow tritone descent (D4→Ab2) + long fade — feels like a process completing, not an explosion
   - Win chime: cold clinical pings (Ab4/D5, "printer confirming") + tritone drone swell — deliberately not triumphant; the hold is filed but Lethe Corp. is still out there
6. **Viewport scaling + ticker fix**:
   - Game was rendering at a fixed 640px width regardless of window size — looked tiny on large screens
   - Fixed via `fitToScreen()`: computes `scale = min(viewportWidth/642, viewportHeight/wrapHeight)` and applies as a CSS `transform: scale()` on `#wrap` (centered, no game-logic/coordinate changes needed), recalculated on load and `resize`
   - Bottom news ticker (`.ticker-wrap`) was clipping its own text — height was 14px against a 14px line-height with no margin; bumped to 20px/20px

## Key code landmarks (index.html)

- `#wrap` — outer container, scaled via `fitToScreen()`
- `fogCvs` / `fogCtx` — offscreen canvas for the darkness/FOV overlay, composited with `destination-out`
- `noiseCvs` / `noiseCtx` — offscreen canvas for the thermal noise texture; drawn with `drawImage` (not `putImageData`, which ignores alpha compositing and previously caused an all-black-canvas bug)
- `AC`, `masterGain`, `droneOsc` — Web Audio nodes; `AC` is lazily created in `initAudio()` on first user interaction (`startGame()`) to satisfy browser autoplay policy
- `MUSIC` — object with `patrol`/`alert`/`chase` patterns (bass + lead note arrays, tempo, volumes)
- `getMusicState()` — reads `erasure.aware` to pick which `MUSIC` pattern is active
- `playChime(type)` — `'die'` or `'win'` end-state stingers, independent of the loop music
- `BOOT` — array of `[delay, class, text]` tuples driving the terminal boot sequence on load
- Game state machine: `idle` → `playing` → `ending` (legal hold filed, ERASURE-7 reacting) → `win`, or `playing` → `die`
- `document.visibilitychange` listener suspends/resumes `AC` when the tab is hidden

## Roadmap (not yet implemented)

- [ ] Multi-floor system (Executive suites → Open office → Server room)
- [ ] COMPLIANCE-2 second enforcement unit (amber, wider FOV)
- [ ] Further Lethe Corp. boot screen / death/win UI polish
- [ ] Level progression (5–10 maps), discussed but not built
- [ ] Lure mechanic, discussed but not built

## Known constraints / gotchas

- All game coordinates are tied to a fixed 640×480 canvas (`CW`, `CH`, `TILE=32`, `COLS=20`, `ROWS=15`) — the viewport scaling fix uses CSS transform precisely to avoid touching this
- `AudioContext` must be created inside a user-gesture handler (`startGame()`), not at page load
- Preview tool's iframe can report very small `window.innerWidth` (seen as low as 4px) — not representative of a real browser window; verify scaling logic by reasoning about the formula rather than trusting preview screenshots at face value
