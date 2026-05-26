# Palette derivation

The skill resolves brand palette values from two sources:

1. **Curated palettes** — one JSON file per offered brand color in `references/palettes/`. The skill loads the chosen file and substitutes its values directly into `design-system.css` and `palette.ts`. No derivation at scaffold time.
2. **Custom hex** — when the user picks "Other (custom hex)" for brand color, the skill runs the hex-mix algorithm below to produce the three `accent*` token values × two modes.

This document defines the algorithm for case 2 and documents the locked values used by both cases for `signal` and `danger`.

## Inputs

- `accent` — user-supplied or curated hex (e.g. `#0891b2`).
- `neutral` — locked to **Slate**. The skill does not ask for this.
- `signal`, `danger` — locked. Same shades in light and dark mode. See "Locked semantic colors" below.

## Slate scale (locked)

| Step | Hex       |
|------|-----------|
| 50   | `#f8fafc` |
| 100  | `#f1f5f9` |
| 200  | `#e2e8f0` |
| 400  | `#94a3b8` |
| 500  | `#64748b` |
| 700  | `#334155` |
| 800  | `#1e293b` |
| 900  | `#0f172a` |
| 950  | `#020617` |

## Output

13 tokens × 2 modes (light + dark). The accent family (3 × 2 = 6 values) is the only part that varies with the user's brand pick.

### Surfaces — driven by `neutral` (locked)

| Token       | Light             | Dark              |
|-------------|-------------------|-------------------|
| `page`      | `#ffffff`         | `slate.950`       |
| `surface`   | `slate.50`        | `slate.900`       |
| `hairline`  | `slate.200`       | `slate.800`       |

Note: `page` light is pure white (cleaner than `slate.50`); `page` dark uses `950` for true blackness.

### Text — driven by `neutral` (locked)

| Token         | Light          | Dark            |
|---------------|----------------|-----------------|
| `ink-body`    | `slate.700`    | `slate.200`     |
| `ink-display` | `slate.900`    | `slate.50`      |
| `ink-muted`   | `slate.500`    | `slate.400`     |

### Accent — driven by user-picked brand color

| Token            | Light                                                | Dark                                                       |
|------------------|------------------------------------------------------|------------------------------------------------------------|
| `accent`         | accent hex                                           | a slightly lighter shade of the accent (e.g. Tailwind 400 for a 600 base) |
| `accent-faded`   | accent mixed 12% on `#ffffff`                        | accent mixed 18% on `slate.950` (`#020617`)                |
| `accent-display` | one step darker than accent (e.g. Tailwind 700 for a 600 base) | one step lighter than dark `accent` (e.g. Tailwind 300 for a 400 dark base) |

`accent-display` is intended for foreground use *on top of* `accent-faded` (e.g. `bg-accent-faded text-accent-display` for a soft badge), where contrast against the lighter background needs more depth than the base `accent` provides.

For curated palettes, all six values are pre-computed and live in `references/palettes/<name>.json`. For custom hex picks, the skill computes `accent-faded` via the mixing formula below, and produces sensible `accent`/`accent-display` shifts (or, for simplicity, can reuse the base hex across both display tokens — the page will still render, just less polished).

### Locked semantic colors — `signal` and `danger`

Both are locked because they carry semantic meaning (warning, danger/error) and shouldn't shift with brand. Same shade in both modes; only the faded backdrop differs.

| Token             | Light                              | Dark                                           |
|-------------------|------------------------------------|------------------------------------------------|
| `signal`          | `#fcd34d`                          | `#fcd34d`                                      |
| `signal-faded`    | `#fffaea`                          | `#2f2b21`                                      |
| `signal-display`  | `oklch(76% 0.165 82)`              | `oklch(93% 0.13 95)`                           |
| `danger`          | `oklch(64% 0.22 25)`               | `oklch(72% 0.20 22)`                           |
| `danger-faded`    | `oklch(96% 0.025 25)`              | `oklch(22% 0.05 22)`                           |
| `danger-display`  | `oklch(55% 0.20 25)`               | `oklch(82% 0.18 22)`                           |

These values are written verbatim into the templates regardless of which brand color the user picks.

## Mixing formula (custom hex only)

Given a foreground hex `F` and a background hex `B`, mixing `F` at percentage `p` (0..1) on `B`:

```
out.r = round(F.r * p + B.r * (1 - p))
out.g = round(F.g * p + B.g * (1 - p))
out.b = round(F.b * p + B.b * (1 - p))
```

Convert each component to a two-digit hex and concatenate with `#`.

### Worked example

Inputs:
- accent = `#4f46e5` → `(79, 70, 229)`
- slate.950 = `#020617` → `(2, 6, 23)`

`accent-faded` light: `#4f46e5` mixed 12% on `#ffffff`:
- r = round(79*0.12 + 255*0.88) = round(9.48 + 224.4) = 234
- g = round(70*0.12 + 255*0.88) = round(8.4 + 224.4) = 233
- b = round(229*0.12 + 255*0.88) = round(27.48 + 224.4) = 252
- → `#eae9fc`

`accent-faded` dark: `#4f46e5` mixed 18% on `#020617`:
- r = round(79*0.18 + 2*0.82) = round(14.22 + 1.64) = 16
- g = round(70*0.18 + 6*0.82) = round(12.6 + 4.92) = 18
- b = round(229*0.18 + 23*0.82) = round(41.22 + 18.86) = 60
- → `#10123c`

These match the values stored in `references/palettes/indigo.json`, confirming the formula.
