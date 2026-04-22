# Studio Base — Visual Identity Template

Generic studio design system for Carlos Bosco's HyperFrames video studio.

**Usage:** Copy this file as `DESIGN.<client>.md` inside a project and customize the tokens per client brand. The `assets/brand-tokens.css` at the workspace root mirrors these defaults — copy and override per project.

---

## How to Customize Per Project

1. Copy this file: `cp DESIGN.studio.md video-projects/<slug>/DESIGN.<client>.md`
2. Copy brand tokens: `cp assets/brand-tokens.css video-projects/<slug>/assets/brand-tokens.css`
3. Update hex values, fonts, and motion rules to match the client's brand guidelines
4. Reference the new file in your composition's `<link rel="stylesheet" href="assets/brand-tokens.css">`

---

## Default Style: Premium Restraint

The default aesthetic is "premium restraint": near-black canvas, monochrome type, fast cuts (~1.5s/scene), motion-blur transitions. Override with client brand where needed. Always keep the *discipline* (one idea per beat, breathing outros) even when adapting the palette.

Reference motion rules: `MOTION_PHILOSOPHY.md` at workspace root.

---

## Colors

| Token | Hex | Role |
|---|---|---|
| `--brand-bg` | `#07090d` | Canvas — near-black |
| `--brand-surface` | `#111318` | Cards, panels, surfaces |
| `--brand-surface-2` | `#1c2028` | Elevated surface |
| `--brand-border` | `#252830` | Borders, hairlines, dividers |
| `--brand-accent` | `#ffffff` | Primary highlight, CTAs — **override with client color** |
| `--brand-accent-2` | `#888888` | Secondary accent |
| `--brand-accent-glow` | `#ffffff` | Glow color (use at 30–75% opacity in `drop-shadow`) |
| `--brand-warn` | `#e0a030` | Use sparingly for contrast moments |
| `--brand-text` | `#ffffff` | Primary text on dark |
| `--brand-text-dim` | `#666666` | Secondary / meta text |

**Replacing a color:** update the hex in `assets/brand-tokens.css` inside the project folder. The 12 sample projects under `video-projects/` use `--ais-*` tokens — leave them as-is (reference only).

---

## Typography

- **Roboto Mono (Medium 500)** — monospace. UI labels, numbers, stats, terminal lines, pill text, CTAs.
- **Montserrat (Light 300 / Bold 700)** — display. Headlines, body copy, taglines.

Pair them always. Roboto Mono labels above Montserrat headlines is the default house pattern.

**Swapping fonts:** update `--font-mono` and `--font-display` in `brand-tokens.css` and load them via Google Fonts `<link>` in `index.html`.

---

## Logo

- No default logo — add client logo as `assets/logo.png` (white / transparent bg)
- CSS glow pattern: `filter: drop-shadow(0 0 50px rgba(255,255,255,0.4));`
- Clearspace: half a logo-height margin on all sides
- Never recolor. Never stretch.

---

## Motion Rules

- **Entrance only:** every element animates in via `gsap.from()`. Transitions handle exits.
- **Easing palette:** `power3.out`, `expo.out`, `back.out(1.4)`, `power4.out` (entrances); `power2.in` (hand-offs); `sine.inOut` (ambient loops)
- **Duration bands:** snap entrances 0.3–0.5s, headline entrances 0.5–0.8s, ambient drifts 2–4s
- **Offset first animation** 0.1–0.3s from scene start
- **Text stagger:** 0.04–0.08s per character (display type), 0.12–0.18s per word (headlines)

---

## Transitions

| Scene change | Transition | Duration | Ease |
|---|---|---|---|
| Standard | Push slide left | 0.35s | `power2.inOut` |
| Opener | Zoom through | 0.35s | `power4.inOut` |
| Outro | Blur crossfade | 0.5s | `sine.inOut` |

---

## What NOT to Do

1. **No full-screen linear gradients** on dark backgrounds — H.264 banding. Use solid `--brand-bg` + localized radial glow.
2. **No `transparent` in gradients** — shader-compatible CSS. Use `rgba(7,9,13,0)`.
3. **No `Math.random()` or `Date.now()`** — render determinism. Use seeded PRNG if needed.
4. **No exit animations** on any scene except the final — transitions handle exits.

---

## File References

- `assets/brand-tokens.css` — CSS `:root` vars imported by every composition
- `MOTION_PHILOSOPHY.md` — gold-standard motion aesthetic (workspace root)
- `DESIGN.studio.md` — this file (generic base)
- `DESIGN.<client>.md` — per-project override (create per project)
