# Roadmap

## Known gaps (not oversights — documented tradeoffs)

- **Hit-zones are bounding boxes, not pixel-perfect masks.** A tap can land
  inside a region's rectangular hit-zone but outside its actual traced
  silhouette (e.g. a corner of the box) and still register. Good enough in
  practice so far; genuine pixel-alpha hit-testing against the mask itself
  would be the next step if mis-taps become a real problem.
- **No cross-tab deep-linking yet.** Profile's Personal Record cards open
  Progress's landing screen rather than jumping straight to that exercise's
  drill-in chart, since each tab manages its own internal sub-navigation
  independently of the shell. Solvable, just not built yet.
- **Muscle-balance status thresholds are a placeholder.** The neglected /
  balanced / overstressed cutoffs (session counts over a 30-day window) are
  a reasonable starting guess, not calibrated against real usage data.
  Worth revisiting once there's actual usage to learn from.

## Exercise library

The 330-exercise library (10 regions × 3 modes) is a strong first pass, not
a final research-grade set. A deeper pass — real exercise selection
research, form cues, progression/regression relationships, video or image
references — is the natural next step once the app itself is stable.

## Explicitly deferred (not forgotten)

- **The bodyweight → equipment "graduation" moment** — after completing a
  full structured no-equipment program (a genuine milestone, not just a
  week or two), offer the choice to continue bodyweight-only or transition
  to equipment-based training.
- **Music** — flagged early, intentionally parked until the rest of the app
  is otherwise finished.
- **A rotating 3D body model** (Tripo/Three.js) — considered and explicitly
  set aside in favor of the flat masked-image approach, mainly on GPU/battery
  cost grounds for a training app used mid-workout. Worth reconsidering only
  as later polish, not a core-experience change.
- **The V3–V6 training-engine concept** (adaptive workout generation,
  progression tracking, phased multi-week programming, physique archetypes)
  — a legitimate and larger future direction, deliberately treated as a
  separate project phase rather than something to fold into the current
  exercise-library work.
