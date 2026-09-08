# Issues & How They Were Solved

This is a living document — quality over completeness. Each entry covers a
real problem that surfaced during development, not a hypothetical one, and
focuses on the *reasoning* behind the fix, not just the diff.

---

## 1. The recurring "stray same-color pixel" bug in muscle mask extraction

**Where it showed up:** three separate times, in three different regions,
during the anatomical mask-extraction pipeline.

**The setup:** to get pixel-precise muscle masks (rather than hand-guessed
ellipses), a flat-color "ID map" reference image was processed with k-means
clustering to find each region's dominant color, then connected-component
analysis to locate every blob of that color.

**First occurrence:** a small cluster of pixels near the ankle straps
happened to match the same k-means bucket as the Obliques/Core color, purely
by coincidence of shading. Because the classification assigned *every* pixel
of that color to Core, those ankle pixels silently became part of the Core
mask.

**Root cause:** color alone isn't a reliable classifier when an image has
enough shading variation that unrelated areas can drift into the same color
cluster. The fix needs *position*, not just color.

**The fix:** added an explicit y-range guard to every region assignment —
Core could only claim pixels within `y: 0.27–0.55` (a defensible anatomical
range for the torso), regardless of what color cluster they belonged to.

**Second occurrence:** the exact same failure mode, on the *back* view,
in a completely different region — Arms had ~6,300 stray pixels near the
ankle. Same root cause, different symptom, only caught because a systematic
blob-position audit was run on every region rather than trusting the one
fix already applied.

**The real lesson:** the first fix patched the one instance that had been
found — it didn't defend against the *pattern*. The second occurrence made
clear this was a systemic risk, not a one-off. The actual fix was to add
position guards to **every single region assignment on both views**, not
just the ones that had already broken, and to re-verify with an automated
check (`arms pixels below y=0.5 should be 0`) rather than eyeballing the
result.

**Verification method:** rather than trust a visual check (which had already
missed this twice), used quantitative connected-component analysis — count
blobs per region before and after the fix, confirm strays are actually gone,
not just less visible.

---

## 2. Chest was untappable — a hit-zone stacking bug, not a data bug

**Symptom:** every muscle on the front-view map was tappable except Chest.

**Investigation:** the mask itself was correct — chest was tinting properly
when triggered manually. The problem was that taps never reached it.

**Root cause:** Shoulders' bounding box (`x: 27.5%–72.5%`) fully contained
both of Chest's bounding boxes. Hit-zones were rendered as absolutely
positioned `<div>`s in list order, and Shoulders was appended to the DOM
*after* Chest — so it sat visually on top (even though invisible) and
silently intercepted every click meant for the smaller region beneath it.

**The fix:** rather than special-case this one overlapping pair, hit-zones
are now sorted by area before rendering — the largest region always renders
first (bottom of the stack), the smallest last (top). This means *any*
future region whose bounding box happens to sit inside a larger neighbor's
box is automatically protected, not just the one pairing that happened to
get caught.

**Lesson:** the bug was invisible by construction — there was no visual
artifact to notice, just a dead tap target. Worth remembering that
overlapping absolutely-positioned hit-zones need an explicit stacking
strategy; relying on DOM insertion order is an accident waiting to happen
the moment two regions' boxes overlap.

---

## 3. A CSS class that was referenced but never defined

**Symptom:** the Muscle Map's FRONT/BACK toggle rendered as plain,
completely unstyled browser buttons — no pill shape, no active state,
nothing.

**Root cause:** the markup referenced `class="toggle"`, but no `.toggle`
CSS rule existed anywhere in the stylesheet. Likely dropped silently during
one of several restructuring passes on the file.

**The fix:** rather than reintroduce a generically-named `.toggle` class
(which had no clear ownership), defined a properly scoped `.mm-toggle` rule
matching the existing pill-segmented-control pattern already used elsewhere
(the duration toggle on Assign Week), and updated the markup to reference it.

**Lesson:** a missing CSS rule fails silently — there's no console error for
"this class doesn't exist," the browser just renders default styles. Worth
a periodic grep-based audit (`grep class="X"` vs `grep "\.X {"`) on a file
this size rather than assuming every referenced class has a matching rule.

---

## 4. A regression introduced by combining multiple files into one

**Context:** the app started as several files (`index.html`, `router.js`,
`db.js`, per-tab view modules) before being consolidated into a single
`ignite.html`, matching the project's single-file philosophy.

**What got lost in the merge:** the original router supported view
`unmount()` functions — a view could return a cleanup callback (for timers,
event listeners) that the router would call before loading the next view.
When the code was inlined into one script, this support was silently
dropped; `loadView()` stopped storing or invoking the return value at all.

**Why it mattered:** the Active Workout view runs a `setInterval` to display
a live elapsed-time timer. Without unmount support, navigating away mid-workout
(e.g. tapping a bottom-nav tab) would leave that interval running forever in
the background — a genuine memory/CPU leak, invisible until someone left the
app open for a while.

**The fix:** restored proper unmount tracking in the router (`currentUnmount`
variable, called and cleared before each new view loads) rather than working
around it locally (e.g. clearing the interval on a `hashchange` listener
inside the view itself). The general mechanism is more valuable than a
one-off patch, since any future view with a timer or subscription benefits
automatically.

**Lesson:** consolidating files is easy to verify for *syntax* (does it
parse) and much harder to verify for *behavior* (does every mechanism the
original files coordinated on still work). This one didn't surface until a
feature was built that actually needed it, months of build-time later.

---

## 5. Reconciling "the model shouldn't be anatomically precise" with "the highlight shouldn't bleed into neighboring muscles"

**The tension:** early muscle-map versions used simple geometric shapes
(ellipses) sized and positioned to roughly match each muscle group. This
was intentional — the model was explicitly meant to be a training *interface*,
not an anatomy textbook. But the same simplicity meant a tap on Chest could
visibly highlight into the upper abs, because an ellipse doesn't know where
a pec actually ends.

**Why the obvious fixes didn't work:**
- Shrinking the ellipse just moved the bleed elsewhere or under-covered the
  real muscle.
- Adding more masking *layers* didn't help — every mask added so far solved
  *outer* silhouette containment (don't let color spill off the body), which
  is a completely different problem from *internal* boundary precision
  (don't let Chest's color spill into Abs).

**The actual fix:** replace the shape-based highlight system entirely.
Instead of computing a `clip-path: ellipse()` per tap, each region got its
own precisely traced alpha mask, and the highlight layer's `mask-image`
swaps to that exact file per tap — no shape math, no approximation. The
boundary is only ever as accurate as the traced mask, which is a
fundamentally different (and much higher) ceiling than shape math.

**Lesson:** when a class of bug keeps resurfacing despite tuning the same
mechanism (tighter ellipse, different blend mode, more blur), it's worth
asking whether the mechanism itself is the wrong tool — not whether it needs
one more adjustment.

---

## 6. Verifying alignment quantitatively instead of trusting a visual glance

**Context:** the mask-extraction reference image (an AI-generated "ID map")
and the actual photorealistic body render it needed to align with were
generated independently, at slightly different pixel dimensions
(935×1682 vs. 941×1672).

**First attempt:** a semi-transparent overlay of both images, eyeballed.
Inconclusive — too much visual noise to judge alignment by eye.

**Second attempt (flawed):** compared Canny edge detection on both images
directly. Produced a discouraging 50.5% match — but this compared *all*
edges in the ID map (including internal muscle-definition lines that have
no counterpart in a flat silhouette) against only the *outer* silhouette
edge of the real render. Apples to oranges — the low score reflected the
comparison being wrong, not the alignment being wrong.

**Correct method:** computed Intersection-over-Union (IoU) on the two
images' outer silhouettes only (binary "is this pixel part of the figure"
masks). Result: 96.6% — genuinely strong alignment, confirming what a
careful visual read had already suggested.

**Lesson:** a bad quantitative test is worse than no test — it produces
false confidence in the wrong direction. Worth designing the metric to
actually match what's being asked ("do the *outlines* line up?") rather
than reaching for the first available comparison function.

---

## 7. Region taxonomy: resolving a genuine trade-off, not just picking a number

**The question:** how many separate muscle regions should the body map
support? A reference chart suggested 11 (splitting things like Abs/Obliques
and Thighs/Calves that an earlier design had merged into single regions).

**The concrete cost of going finer:** prototyping the 11-region split
revealed that Obliques, specifically, became a genuinely too-narrow tap
target — a ~4%-wide strip on a phone screen, realistically prone to mis-taps.

**The resolution:** rather than accept either extreme (stay coarse and
lose anatomical usefulness, or go fully granular and accept a broken tap
target), merged only the one problematic pair — Abs + Obliques into a
single "Core" region — while keeping the other splits (Thighs/Calves,
Arms/Forearms) that tested fine. Final count: 10 regions, not 7 or 11.

**Lesson:** "how many regions" wasn't a single decision made once — it was
tested concretely (a real tappable prototype, not a mockup) before being
finalized, and the final answer was a genuine compromise informed by that
test, not a number picked in the abstract.

---

*(This log will keep growing as real issues surface — intentionally not
padded with minor fixes to look more thorough than it is.)*
