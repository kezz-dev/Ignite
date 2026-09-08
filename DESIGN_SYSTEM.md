# Design System

## Color tokens

| Token | Hex | Use |
|---|---|---|
| `--bg` | `#0B0B0D` | App background — warm off-black, never pure black |
| `--surface` | `#17171A` | Default card surface |
| `--surface-alt` | `#202024` | Nested surface (inside a card) |
| `--hairline` | `#2A2A2E` | All borders |
| `--accent` | `#FF4E1F` | Primary orange — CTAs, active states, key numbers |
| `--accent-hot` | `#FF8A4C` | Lighter orange — gradients, secondary emphasis |
| `--text` | `#F2F0EC` | Primary text — warm off-white, not pure white |
| `--text-muted` | `#8A8A8F` | Secondary text, labels |

**Muscle-balance status colors** (Progress + Body Pulse *only* — never general UI):

| Token | Hex | Meaning |
|---|---|---|
| Blue `#2F8FE0` | Neglected |
| Green `#34C77B` | Balanced |
| Purple `#A855F7` | Overstressed |

Orange is reserved for *action*. Blue/green/purple are reserved for *status*.
The two vocabularies never mix — an orange button never means "neglected,"
and a status color never appears on a button.

## Typography

| Font | Use |
|---|---|
| **Anton** | Big display moments only — workout names in hero cards, exercise titles mid-workout. Always uppercase. A shout, not a whisper — used sparingly. |
| **JetBrains Mono** | Every number, stat, timestamp, badge — and small uppercase eyebrow labels. |
| **Manrope** | Everything conversational — body text, buttons, nav labels. |

Rule of thumb: if it's a number, it's mono. If it's a big emotional
headline, it's Anton. Everything else is Manrope.

## Signature motifs

- **Heat bars** — thin bars filled proportionally with the orange gradient,
  representing training intensity. First used on the workout-in-progress
  exercise list, reused in Home's hero card.
- **Body Pulse** — a small cropped/tinted body thumbnail teasing the muscle
  balance status, living on Home.
- **The masked-highlight technique** — any time muscle color-coding appears
  on the body, it's masked against the real body render's alpha channel so
  color can never spill past the silhouette. See `ISSUES.md` for how this
  evolved from shape-based approximation to precisely traced masks.

## Anti-patterns (deliberately avoided)

- No red — reserved conceptually for errors, never for muscle color or
  general accents (this is why the anatomy base render is gray, not red).
- No pure white or pure black — both the background and primary text use
  warm off-tones, easier on the eyes for long sessions on OLED screens.
- No orange used for status meaning, no status color used for actions.
- No flat, static cards for anything meant to feel alive (hero cards, Body
  Pulse) — these carry some gradient/glow quality. Flat fills are reserved
  for plainly informational surfaces (settings rows, session logs).
