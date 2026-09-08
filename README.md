# 🔥 Ignite

**A workout companion built around one idea: your body should be the interface.**

Ignite is a single-file, offline-first Progressive Web App for tracking workouts, built around a tappable anatomical muscle map instead of a conventional exercise-list-first design. Tap a muscle, see how to train it. Log a session, watch your actual training balance show up on the same body you tapped.

---

## What makes it different

Most workout apps are exercise databases with a body diagram bolted on for decoration. Ignite inverts that: **the body is the primary interface**, and everything else — the exercise library, the program calendar, the progress tracking — exists to serve what happens when you tap it.

A few things that came out of that decision, each a deliberate choice rather than a default:

- **Pixel-precise muscle masks, not shape approximations.** Early versions highlighted a tapped muscle with a guessed ellipse — which meant Chest's highlight visibly bled into the abs, because an oval doesn't know where a pec actually ends. The final version traces each of 10 muscle regions from a hand-verified anatomical reference image, so the highlight *is* the muscle's real outline. See `docs/ISSUES.md` for the full story of how that got built.
- **A weighted heat map, not a flat counter.** A bench press doesn't train chest and shoulders equally, so Ignite's muscle-balance system doesn't count them equally either. Every exercise carries per-region contribution weights (chest: 1.0, shoulders: 0.5, for example), and "how trained is this muscle" is a genuine weighted sum, not a naive tally of "did you touch it."
- **Honest empty states, everywhere.** A fresh install shows "no data yet" — never a fabricated "you're neglecting your arms" message with nothing behind it. If Ignite doesn't know something about you yet, it says so.
- **Diagnose, then prescribe.** The muscle balance view doesn't just tell you a region is neglected — tapping it surfaces real exercises for that exact muscle, pulled from the same library the muscle map itself uses. The color tells you what's wrong; the tap tells you what to do about it.
- **Equipment-aware from the ground up.** Programs, exercises, and even the app's own onboarding adapt to whether someone has a gym, a few household objects, or just their bodyweight — treated as a real fork in the app's logic, not a filter bolted on afterward.

## Tech stack

Deliberately boring, on purpose: **vanilla JavaScript + IndexedDB, zero frameworks, zero build step.** The entire app is one HTML file (`ignite.html`) — CSS, the data layer, the router, and all four tab views are inlined, matching the single-file offline-first pattern used across the author's other apps.

| Piece | How it's done |
|---|---|
| App shell | One `index.html`-equivalent (`ignite.html`), one `#view-container` |
| Routing | Hash-based, ~40 lines, swaps which tab's `render()` fills the container |
| Data | IndexedDB, 6 object stores (see `docs/ARCHITECTURE.md`) |
| Offline support | Service worker registered from a `Blob` URL (no separate file) |
| Install metadata | PWA manifest generated at runtime, attached via a `data:` URI |
| Muscle map assets | 15 hand-verified alpha masks (front + back, ~415KB total), embedded as base64 |

No React, no Vue, no bundler. It opens in a browser and it works.

## Features

**Home** — Today's session (or "pick up where you left off" if no plan exists), a live consistency ring, a rolling 7-day calendar strip, and a "Body Pulse" widget teasing your current muscle balance.

**Workouts** — Equipment setup (gym / bodyweight), program templates (Push/Pull/Legs, Upper/Lower, Muscle Group Split, Calisthenics, Full Body, or fully custom), drag-and-drop weekday assignment, and the tappable **Muscle Map** itself — front and back, 10 regions, real exercises pulled from a 330-exercise library.

**Progress** — Real consistency history, a muscle-balance summary that opens into the full front/back masked visualization (blue = neglected, green = balanced, purple = overstressed), and per-exercise strength tracking with an actual chart built from logged sets.

**Profile** — Editable name, real personal records computed from logged history, an achievement system with genuine unlock logic, and settings.

**The logging loop that ties it together** — Start (or repeat) a workout from Home, log real sets and weights, finish, and watch the streak, the balance colors, and your personal records all update from that one action.

## Design language

Midnight black background, fiery orange accent — but more specifically: warm off-black (`#0B0B0D`) rather than pure black, `Anton` for big display moments, `JetBrains Mono` for every number, `Manrope` for everything conversational. Blue/green/purple are reserved *only* for muscle-balance status and never used as UI accents, so their meaning stays unambiguous. Full reference in `docs/DESIGN_SYSTEM.md`.

## Running it

```bash
# ES modules and the service worker both require a real origin — file:// won't work
python3 -m http.server 8000
# then open http://localhost:8000/ignite.html
```

## Project status

All four tabs are wired to real data and a real logging flow exists end-to-end. Known open items are tracked in `docs/ROADMAP.md` — most notably, the exercise library's 330 entries are a strong first pass, not a final research-grade set, and a couple of interaction details (pixel-perfect hit-testing vs. bounding-box hit-zones, cross-tab deep-linking) are documented simplifications rather than oversights.

## Documentation

- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — data schema, file structure, how the pieces fit
- [`docs/ISSUES.md`](docs/ISSUES.md) — real problems hit during development and how they were actually solved
- [`docs/DESIGN_SYSTEM.md`](docs/DESIGN_SYSTEM.md) — color tokens, typography rules, signature visual motifs
- [`docs/ROADMAP.md`](docs/ROADMAP.md) — what's next, what's intentionally deferred
