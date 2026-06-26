# Save the Nest — Project Context

A tap-based educational mini-game for young children that teaches **long vs. short**.
The child rebuilds a bird's nest by tapping the correct (longer / shorter) item.

- **Single self-contained file:** [`index.html`](index.html) — vanilla HTML/CSS/JS, no frameworks/build step. Just open it in a browser.
- **Assets:** real art in [`assets/`](assets/) — all raster images are **WebP** (stills + animated bird WebPs; converted from PNG/GIF at quality 90, originals removed), plus SVG for the play button and transition leaves. Background music in [`audio/`](audio/).
- **Design space:** authored at a fixed **1920×1080** stage (`#stage`) that JS scales + centers to fit any screen. A portrait "rotate your device" prompt shows on portrait screens.

---

## Game flow

1. **Title / cover screen** (`#intro`) — `assets/tittle screen.png` cover art + a round **PLAY** button (`assets/play button.svg`) with a pulse animation + click SFX.
2. Tap Play → **leaf transition** (green leaves sweep in to cover the screen, scene swaps behind them, leaves sweep back open) → **Tutorial**.
3. **Tutorial** (Level 1, `teach:true`) — opening beat: the elephant appears **alone** (board + twigs hidden) and greets — "Let us start fixing the nest!". After the line is read, the board springs open, the twigs appear inside it, then **2 guided steps** (tap longer → tap shorter). A wrong tap doesn't fail; it just lets the child retry.
4. Tutorial ends → **both twigs fly off the board and settle into the nest** (`flyPairToNest`, woven copies stay), the **nest grows a little** (`scale(.52)`), the elephant says **"Now, it is your turn!"**, then the **same leaf transition** carries the child into the real game.
5. **Levels 2–6** — the real game, difficulty ramping from clear to very subtle length differences. Each correct answer grows the nest + fills the progress meter. No character dialogue on a correct answer.
6. **Finale** — the last answer's pew-pew fly-by leaves the screen first, THEN the celebration: panel + meter fade out, win melody + confetti, and the three bird GIFs fade in and slowly orbit the nest (`spawnWinBirds`). The **Play Again** art button (`play again.svg`) pops in ~3.2s later, bottom-right, with a gentle pulse (no win card).

### Levels (the `LEVELS` array)
| # | id | items | ask | notes |
|---|----|-------|-----|-------|
| 1 | `tutorial` | long & short **twig** | — | 2 steps: "Tap the long twig." → "Tap the short twig."; then both twigs fly into the nest. `teach:true` |
| 2 | `feather-feather` | long blue feather vs short **red** feather (`small feather.webp`) | `long` | clear difference |
| 3 | `twig-feather` | long twig vs short feather | `short` | clear difference |
| 4 | `feather-leaf` | long feather vs short leaf | `long` | subtler — the leaf is fatter but shorter |
| 5 | `leaf-leaf` | long leaf vs short leaf | `short` | very small difference |
| 6 | `feather-twig` | long feather vs short twig | `long` | hand-tuned `fixed` layout (twig pinned top) |

Items shuffle per play round (answer isn't positional). The tutorial keeps a fixed layout (short on top, long below) with hand-tuned `top` overrides (small twig 256.5px, long twig 698px). Play-level banner prompts come from `buildPrompt()` — "Tap the LONG item!" / "Tap the SHORT item!" per the level's `ask` (asks alternate long/short across levels 2–6). Levels are asset-gated by `resolveLevels()` — a level is skipped if its art fails to load.

---

## Screen layout (current)

- **Background:** `assets/background.png` (tree branch + sky), full-bleed (`#stage .bg`). Ambient falling leaves (`#leaves`) drift over it.
- **Question banner** (`#qbanner`) — **top-center**, `question box.png` frame sized 1000×149 to the art's natural aspect, 44px text. Shown in play levels; hidden in the tutorial (instruction lives in the speech bubble instead).
- **Nest** (`#nest`) — center, resting on the branch fork. Grows from `scale(0.42)` → `1.0` as the nest is built. The fuller `big nest.png` swaps in FULLY past the halfway point (never held at partial opacity — that read as a ghostly shadow); `dens nest.png` fades in on the final level.
- **Progress meter** (`#progress`) — **vertical bar on the left**, fills bottom→top, blue gradient. The `dens nest.png` image rides the top of the fill.
- **Item panel** (`#panel`) — **right side**, CSS-drawn cream board with a glossy wooden/orange frame (`.panel-frame` + `.panel-inner`). Holds the two tappable items in `#slot0` (27%) / `#slot1` (72%). **Springs open** (`panelOpen`) at the tutorial's start — tutorial only; play rounds don't re-open it. Geometry: `left:1612, top:150, 309×780` (right edge 1921 — 1px clipped, invisible); item widths in `LEVELS` are sized to this board.
- **Elephant character** (`#mascot`) + **speech bubble** (`#dialogue`) — bottom-left. Used to guide/cheer in the tutorial and to explain a wrong tap in Level 2.

---

## Key mechanics & feel

- **Tutorial reveal (after the transition):** elements appear **one by one, slowly** — elephant → twig → twig → **then** the speech bubble → the correct-twig hint.
- **Tutorial pacing:** renderRound puts `.teach` on `#stage` during the tutorial — CSS overrides slow every animation a touch (entrance 1.15s, shake .85s, bubble .8s, etc.), the reveal delays stretch, the typewriter runs at `TEACH_TYPE_MS = 70`, and the twig flight glides at 1.9s (vs 1.5s in play).
- **Speech bubble = typewriter:** the elephant's lines type out letter-by-letter (`TYPE_MS = 60`ms/char), then wait a reading pause (`READ_PAUSE = 1600`ms) before anything advances. Preserves highlighted words + line breaks.
- **Correct-twig hint (tutorial):** the twig the instruction asks for **pulses** (scale `itemNudge`). No glow. Clears on a correct tap; re-appears after a wrong tap.
- **Correct tap (play):** star pop + a fly-by bird (`flying bird 1/2/3.webp`, says "pew pew"; bird 1 is mirrored), and **only the correct item** glides into the nest (`flyToNest`, slow 1.5s arc) — it fades as its woven copy appears, and the nest grows + meter fills as it lands. The other item stays on the board until the next round. No "Long/Short" label, no dialogue.
- **Wrong tap (all play levels 2–6):** the item shakes + error sound (no red glow) and the question banner **types out** "Oops! That is not right." via the `typeText` typewriter (`bannerOops()` — no banner pop; "Oops!" coloured via `#qtext .oops`; the `.tw-c` reveal classes are shared with the speech bubble). The prompt is restored after a read pause; a correct tap clears it immediately (guarded by `oopsToken`). No elephant wrong-tap explanation. The tutorial keeps its own guidance (pulse the correct twig; banner hidden there).
- **"Long"/"Short" labels:** shown **only in the tutorial**, below each twig — a label appears **only when its twig is tapped** (never pre-shown) and stays until the twigs fly into the nest.
- **Idle nudge:** items gently pulse after ~8s of inactivity in **play levels only** (disabled in the tutorial).
- **Audio:** background music loops (low volume) + synthesized WebAudio SFX (tap/correct/wrong/star/win/pew/click). Started on first user gesture. There is **no** sound toggle button.

---

## Notable constants (top of the script)
- `IDLE_MS = 8000` — idle before the behavioral nudge.
- `NUDGE_AFTER_WRONG = 4000` — nudge sooner after a mistake.
- `TYPE_MS = 60` / `READ_PAUSE = 1600` — typewriter speed + post-line reading pause.
- `NEST_MIN = 0.42` — nest start scale (grows to 1.0).

## Key functions
- `leafTransition(onCovered)` — the leaf cover/reveal wipe (uses `leaf-0…4.svg` + a CSS-transition green backdrop). Called on Play and at tutorial-end.
- `renderRound(slow)` — builds a round; `slow=true` does the staggered one-by-one tutorial reveal.
- `handleTap(item)` — tutorial vs. play branching, correct/wrong handling. Items are wired via `attachInput` on the standard `click`/`touchend` events (not pointerup — that was unreliable on some devices); the child `<img>` is `pointer-events:none` so taps land on the `.item` div.
- `guide(inner, big, onTyped)` — elephant speech in the tutorial (typewriter via `typeText`). (The old `showCharacter` wrong-tap explainer was removed — wrong taps now use `bannerOops()`.)
- `pulseCorrect()` / `clearPulse()` — tutorial correct-twig pulse hint.
- `advanceBuild()` — grows the nest + fills the meter on a correct play answer.
- `flyToNest(item, delay, onLand)` — slow (1.5s) arc into the nest bowl (`nestTargetPoint()` accounts for the nest's grow-scale); the item fades as its woven copy appears. Play rounds fly only the correct item; `flyPairToNest(leadItem)` flies BOTH twigs at tutorial-end.
- `spawnWinBirds()` / `clearWinBirds()` — win celebration: 3 bird GIFs orbit the nest on sampled-keyframe ellipses (own pace/phase, smooth turn at the extremes); waits for the last pew-pew fly-by (`flybirdEndsAt`).
- `fit()` — scales/centers the 1920×1080 stage to the viewport.

---

## Assets in use ([`assets/`](assets/))
- **Backgrounds/UI:** `background.webp`, `tittle screen.webp`, `question box.webp`, `play button.svg`, `play again.svg`.
- The folder is now trimmed to only files the game references (the unused `Feather big`, `long feather`, `object placeholder`, `progress bar` were removed).
- **Nest:** `small nest.webp`, `big nest.webp`, `dens nest.webp`.
- **Items:** `long twig.webp`, `small twig.webp`, `long feather 2.webp` (blue), `small feather.webp` (red), `Long leaf.webp`, `short leaf.webp`.
- **Character:** `character.webp`, `character dialouge box.webp`.
- **Transition leaves:** `leaf-0.svg` … `leaf-4.svg`.
- **Birds:** `flying bird 1/2/3.webp` (animated WebP).
- **Audio:** [`audio/Background music.mp3`](audio/).
- The item panel and progress track are **drawn in CSS** (not images).

---

## Verification workflow
Changes are checked headlessly with a temporary Puppeteer install:
`npm install --no-save puppeteer`, load the file via a `file://` URL with `--allow-file-access-from-files`, take screenshots / read the DOM, then **remove all dev artifacts** (`node_modules`, scratch `.js`, screenshots).
