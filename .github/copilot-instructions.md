# Zenquarium — Copilot Instructions

Zenquarium is a premium, meditative aquarium simulator. Single HTML file (`index.html`, ~7,700 lines), zero dependencies, runs entirely in the browser.

## Running

Open `index.html` directly in a browser. No build step, no server required.

## Architecture

The file is organized as a sequence of revealing-module IIFEs, each exposed as a `const`:

```
Utils        — pure math/DOM helpers
ZenStorage   — LocalStorage read/write wrapper
ZenAudio     — Web Audio API synthesis (no audio files)
Themes       — theme config objects (fish types, palettes, audio params)
Fish (class) — individual fish entity: SVG rendering + trait-based AI
Tank         — canvas environment (background, plants, bubbles, lighting)
Creatures    — jellyfish and octopus/squid entities
Achievements — unlock tracking and one-time bonus logic
UI           — DOM panels: splash, HUD, shop, modals, milestone overlays
App          — game loop, scoring engine, state machine
```

Dependency order matters — each module references the ones above it. New code follows the same IIFE pattern.

## Key files

| File | Purpose |
|---|---|
| `zenquarium-spec.txt` | Master design doc (~34 KB). Authoritative source for all game rules. |
| `zenquarium_prompt.txt` (parent dir) | Original requirements (~25 KB). Context for design intent. |
| `scoring-rules.md` | Scoring formulas, multipliers, bonuses, and penalties. Consult before changing any numbers. |
| `plan.md` | Active roadmap (Phases 1–4): pointer input, session snapshots, mobile packaging, retention. |

**Always check `zenquarium-spec.txt` and `scoring-rules.md` before modifying scoring, creatures, fish behavior, or theme definitions.**

## Conventions

- **No external dependencies** — never add a CDN link, import, or network call.
- **Canvas for gameplay, DOM for UI** — fish, tank, creatures draw on `<canvas>`; HUD, modals, shop use DOM/CSS.
- **SVG fish rendering** — each fish is drawn as inline SVG with species-specific markings (stripes, spots, fin shapes). Fish types are defined in the `Themes` config; draw routines are in the `Fish` class.
- **Config at the top of each module** — magic numbers live in the theme config or a local `CONFIG`/`PARAMS` object, not scattered inline.
- **Audio is synthesized** — all sounds use `AudioContext`, `OscillatorNode`, `BiquadFilterNode`, etc. Do not add `<audio>` tags or static audio files.
- **LocalStorage only** — no IndexedDB, no cookies, no server. Key prefix: `zenquarium_`.
- **Offline-first** — the app must work with no network connection.
- **Responsive + touch** — use pointer events; canvas scales to fill the viewport.

## Scoring system

- Base rate: +0.5 pts/sec per healthy fish × multiplier
- Multiplier streak (all fish >80% health): ×1 → ×1.5 (30 s) → ×2 (90 s) → ×3 (180 s+)
- 20 one-time achievement bonuses (+25 to +500 pts)
- Repeatable bonuses: Perfect Tank +25/60 s, Biodiversity +15/45 s, Clean Sweep +20
- Penalties: fish death −15, starvation −20, provoked stingers −25, neglected tank −10/30 s
- Action costs: Feed −5, Clean −10, Buy Fish −15 to −30
- 5 milestone tiers pause the game and offer a quit option: Bronze 1k, Silver 2.5k, Gold 5k, Platinum 10k, Master (15+ fish + all 4 species + 3 min playtime)

## Themes

Five themes, each with unique fish, palette, plants, and ambient audio:

| Theme | Fish types | Visual identity |
|---|---|---|
| Koi Garden | Kohaku, Showa, Ginrin, Tancho | Warm sunset, cherry blossom particles |
| Tropical Reef | Clownfish, Blue Tang, Angelfish, Neon Tetra | Vibrant coral reef |
| Deep Ocean | Anglerfish, Lanternfish, Viperfish, Barreleye | Dark, bioluminescent glow |
| Zen Garden | Golden Carp, Silver Carp, Ink Carp, Ghost Carp | Monochromatic, ink-wash |
| Pen & Ink | (all fish as ink strokes) | Light parchment background (inverted palette) |

## Fish AI

- 8 personality traits: playful, lazy, shy, explorer, brave, curious, social, greedy
- Each fish instance has trait intensities (0.7–1.3×) randomized at spawn
- Movement: velocity lerp toward a randomly chosen target; retarget when close or after timeout
- Schooling: 3+ same-type fish within radius form a loose school
- Hunger/health decay over time; death triggers score penalty
- Population cap: `max(15, min(20, 20 × viewportW × viewportH / (1200 × 800)))`

## Creatures

- **Jellyfish**: passive drift, kills fish on contact, provoked by 3 taps (stingers deploy, then flee)
- **Octopus/Squid**: active hunter, eye tracking, lunge attack, provoked by 3 taps (ink burst, then flee)
- Each theme has species-specific variants (Moon Jelly, Blue-Ring Octopus, Crystal Jelly, etc.)

## Commits

All commits in this repo include:
```
Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
```
