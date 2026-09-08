# Architecture

## File structure

Everything lives in one file, `ignite.html`, structured internally as:

```
<style>   All CSS — design tokens as CSS custom properties, then per-view rules
<body>    #app > #view-container (empty, filled by the router) + fixed bottom nav
<script>  One IIFE containing, in order:
            1. Manifest generation (data: URI)
            2. Service worker registration (Blob URL)
            3. Data layer (IndexedDB wrapper + all read/write functions)
            4. Embedded assets (two body-render images, 15 region masks,
               the 330-exercise library — all as base64/JSON constants)
            5. Shared helpers (REGION_LABELS, TYPE_MUSCLE_GROUPS, etc.)
            6. Each view's render() function (Home, Workouts, Progress, Profile,
               Active Workout)
            7. The views registry + router
```

## Why one file

Matches the offline-first, single-file pattern used elsewhere in the author's
other apps. The tradeoff: file size is large (~3.8MB) because two photorealistic
body renders and 15 alpha masks are embedded as base64 rather than fetched
separately. Every byte of that is visual asset data, not code bloat — the
script itself is a few hundred KB.

## The router

Deliberately tiny (~40 lines). Hash-based: `#home`, `#workouts`, `#progress`,
`#profile`, `#activeWorkout`. Each route maps to a view's `render(container)`
function. A view can optionally return an `unmount()` function for cleanup
(timers, listeners) — the router calls it before loading the next view. See
`ISSUES.md` for why this needed to be *restored*, not just added.

Sub-navigation *within* a tab (Workouts' Equipment Setup → Choose Program →
Assign Week, or Muscle Map's front/back toggle) is never routed through the
shell. Each tab's `render()` manages its own internal state and re-renders
its own container — the router and the shell have no idea those sub-screens
exist. This keeps the shell genuinely dumb and each tab's complexity contained.

## Data layer (IndexedDB, 6 stores)

| Store | Purpose | Key detail |
|---|---|---|
| `profile` | Single record (id=1) | name, equipmentAccess, memberSince |
| `program` | Single record (id=1), the active calendar plan | weekdayPattern, duration, expiresAt |
| `exercises` | The 330-exercise library | Indexed by `primaryRegions` (multiEntry) |
| `workoutLogs` | Every *completed* session | Only ever written by `logCompletedWorkout()` — never for scheduled/planned sessions |
| `achievements` | One-time unlock events | `unlockedAt` stays `null` until earned, then never re-checked |
| `appState` | Small key/value bucket | Currently just tracks first-run seeding |

**The one rule that matters most:** `workoutLogs` entries only exist for
workouts someone actually finished. This is what makes the muscle-balance
heat map trustworthy — it reflects real training, never the plan.

### Muscle group weighting

Each exercise carries a `heatMapContribution` object, e.g.:

```js
{ chest: 1.0, shoulders: 0.5, arms: 0.4 }
```

`logCompletedWorkout()` looks up each performed exercise's weights and sums
them into that log's `regionalLoad`. `getMuscleGroupCounts(days)` then sums
`regionalLoad` across recent logs — so a month of pure bench pressing
correctly shows Chest as more trained than Shoulders, rather than both
counting equally just because both got "touched."

## The muscle map assets

10 regions (front: chest, shoulders, arms, forearms, core, thighs, calves;
back: shoulders, back, lower_back, glutes, arms, forearms, thighs, calves —
6 regions like shoulders/arms/forearms/thighs/calves appear on both views).

Each region is a real traced alpha mask, not a hand-drawn shape — see
`ISSUES.md` for the extraction pipeline (k-means color clustering,
connected-component analysis, position-guarded classification, edge
smoothing). The same 15 mask files power both the single-region tap
highlight (Workouts → Muscle Map) and the all-regions-at-once status
overlay (Progress → Muscle Balance) — one asset set, two renderers.
