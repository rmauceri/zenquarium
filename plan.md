# Zenquarium Mobile App Publishing Plan

## Objective

Prepare Zenquarium (single-file web app) for reliable release in Apple App Store and Google Play with strong mobile UX, stable runtime behavior, performance headroom, and retention loops.

## Guiding approach

1. Stabilize core interaction and lifecycle first.
2. Make session continuity durable across app backgrounding/termination.
3. Hit performance and battery budgets on mid/low devices.
4. Package and comply for both stores.
5. Layer engagement systems after the foundation is reliable.

---

## Area 1 — Mobile Interaction and Lifecycle Reliability

### Goals
- Make gameplay input consistent across touch, stylus, mouse, and trackpad.
- Prevent hidden-tab/background simulation drift and audio issues.
- Ensure predictable pause/resume behavior on calls, app switching, and lock screen.

### Current observations
- Input handling is primarily `click`-based (`#tank`, controls, overlays).
- No explicit `pointer*`/`touch*` handlers for core gameplay.
- No `visibilitychange` / `pagehide` lifecycle hooks currently wired in app loop.

### Work plan
1. **Unify input model**
   - Introduce pointer-first handlers (`pointerdown`, optional `pointermove`) for tank interactions.
   - Keep keyboard/click fallback for desktop browsers.
   - Add press-duration threshold if needed to separate “tap glass” vs “drop food” interactions.
2. **Lifecycle event framework**
   - Add app lifecycle controller in `App` module with events:
     - `visibilitychange`
     - `pagehide` / `pageshow`
     - optional `blur` / `focus`
   - On background: auto-pause simulation and ambient scheduling.
   - On foreground: controlled resume with timestamp reset to avoid `dt` spikes.
3. **Audio robustness**
   - Explicitly suspend audio context on background and resume on user reactivation where required.
   - Keep splash/game ambient transitions deterministic after interruptions.
4. **Validation matrix**
   - iOS Safari + Android Chrome + desktop browsers:
     - call interruption
     - lock/unlock
     - home/app switch
     - orientation change mid-session

### Deliverables
- Input/lifecycle architecture update in `index.html`.
- Regression checklist for tap/feed/creature interactions and pause state correctness.

---

## Area 2 — Session Persistence and Recovery

### Goals
- Preserve active session state (not just meta stats) after app is backgrounded or killed.
- Enable “resume where you left off” safely.
- Keep save format versioned and migration-friendly.

### Current observations
- `ZenStorage` persists high score/sessions/achievements/stats.
- Runtime tank state (fish roster, food, cleanliness, timers, active creatures) is not fully recoverable as an in-progress session.

### Work plan
1. **Define session snapshot schema**
   - `schemaVersion`
   - tank state: score, health/cleanliness, costs, multiplier/streak, mode flags
   - fish array (minimal deterministic fields needed to resume)
   - food, creature state, cooldown timers
   - session metadata (`savedAt`, elapsed runtime)
2. **Autosave policy**
   - Periodic autosave (e.g., every 10–20 seconds).
   - Save on lifecycle transitions (`visibilitychange`, `pagehide`).
   - Save immediately on major state transitions (buy fish, milestone, game-over).
3. **Resume strategy**
   - On launch/start, detect recent valid snapshot and offer:
     - “Resume Session”
     - “Start New Session”
   - Include snapshot expiry rule to avoid stale/confusing recoveries.
4. **Data safety**
   - Add strict load guards and fallback-to-default when malformed.
   - Include migration function per `schemaVersion`.

### Deliverables
- `ZenStorage` expanded with active-session snapshot API.
- Resume modal and restoration flow in `UI` + `App`.
- Backward-compatible migration guardrails.

---

## Area 3 — Performance, Thermal, and Battery Budget

### Goals
- Maintain smooth play on mid/low mobile hardware.
- Reduce battery drain from continuous animation/audio.
- Avoid runaway DOM/animation growth in long sessions.

### Current observations
- Heavy layered rendering (SVG + div particles + ripple effects + animated overlays).
- Frequent dynamic DOM insertion/removal for ripples/effects.
- No explicit “reduced effects” or dynamic quality tier.

### Work plan
1. **Instrumentation pass**
   - Add lightweight FPS and frame-time sampling (debug toggle only).
   - Track counts for fish, particles, bubbles, active creature effects.
2. **Quality tiers**
   - Define `high / balanced / battery` render presets:
     - cap particle/bubble spawn rates
     - reduce ripple count and expensive blur usage
     - tone down shadow/glow effects on low tier
3. **Animation optimization**
   - Audit repeated layout-triggering style writes.
   - Batch DOM updates where practical.
   - Reduce timers/RAF loops running when hidden or paused.
4. **Long-session stability**
   - Add sanity caps/cleanup checks for transient effect nodes.
   - Run soak test scenarios (10–20+ min) on iOS and Android devices.

### Deliverables
- Performance profile report and optimized defaults.
- User-facing “Battery Saver” toggle (optional) mapped to quality tier.

---

## Area 4 — Packaging, Compliance, and Store Readiness

### Goals
- Convert the web app into shipping binaries for iOS/Android.
- Satisfy app-store policy requirements and metadata completeness.
- Ensure predictable offline/online behavior and legal coverage.

### Work plan
1. **Packaging path**
   - Wrap with Capacitor (recommended) for iOS/Android builds.
   - Keep game logic single-source in `index.html`.
2. **Platform assets/config**
   - App icons and splash assets (all required densities).
   - Orientation policy decision and safe-area checks.
   - App name, bundle IDs, versioning strategy, build numbers.
3. **Web-app assets (still useful in wrapper/web)**
   - Add manifest + service worker strategy as needed.
   - Decide offline behavior (full offline supported vs limited).
4. **Compliance**
   - Privacy policy + data disclosure text.
   - Age rating questionnaire prep.
   - Permission minimization (no unnecessary permissions).
   - Third-party SDK policy review if analytics/crash tools are added.
5. **Store listing package**
   - Screenshots/video captures per device class.
   - Description, keywords, localization plan (phase 1+).

### Deliverables
- Buildable iOS/Android projects with release configs.
- App-store submission checklist and artifact list.

---

## Area 5 — Retention and Engagement Systems

### Goals
- Increase return rate without harming “zen” tone.
- Add optional structure for short sessions and progression.
- Preserve premium, low-friction feel.

### Work plan
1. **Daily/Session goals**
   - Light-touch rotating objectives (e.g., maintain biodiversity for 2 min, avoid fish loss for X min).
   - Reward with cosmetic badges/themes/status effects, not intrusive currencies.
2. **Micro-events and seasonal variants**
   - Controlled random events with clear readability and bounded risk.
   - Theme-specific event variants to encourage replay.
3. **Progression layer**
   - Collection/logbook (species seen, creatures observed, challenge history).
   - Milestone tracks beyond raw score for non-competitive players.
4. **Re-engagement hooks (opt-in)**
   - Optional local reminders (mobile notifications) for “tank check-in.”
   - Resume nudges after abandoned sessions.
5. **A/B-ready configuration (later)**
   - Keep event frequencies and reward values configurable in one place.

### Deliverables
- Engagement feature spec with tuneable constants.
- First implementation slice integrated with achievements/status messaging.

---

## Cross-cutting consistency fix to include early

- Resolve scoring mismatch: `Buy Fish` cost differs between docs and runtime (`15` vs `20`).
- Establish one canonical source for balancing constants (inline config object) and derive UI/docs from it.

---

## Suggested phased rollout

### Phase 1 (Foundation)
- Area 1 + lifecycle parts of Area 2 + scoring consistency fix.

### Phase 2 (Durability)
- Full Area 2 recovery flow + Area 3 optimization baseline.

### Phase 3 (Ship prep)
- Area 4 packaging/compliance and submission readiness.

### Phase 4 (Growth)
- Area 5 engagement systems with telemetry-informed tuning.

---

## Execution checklist (high level)

- [ ] Define canonical game-balance config (costs/bonuses/penalties).
- [ ] Implement pointer-first input abstraction.
- [ ] Add lifecycle pause/resume/save hooks.
- [ ] Add active-session snapshot + migration.
- [ ] Add resume UX and conflict-safe restore.
- [ ] Run performance profiling and apply quality tiers.
- [ ] Build iOS/Android wrappers and complete store assets.
- [ ] Add initial retention loop (daily/session goals).

