---
name: design-system
description: Codifies your approved design (a Claude Design mockset, or quick brand picks) into a central style file, reusable UI primitives, and a live reference page for your application.
---

# Design System Builder

You are guiding a builder through scaffolding a complete design system into a React + Tailwind v4 codebase. The output is a single-page reference at `/admin/design-system` that previews and documents every primitive, plus a small set of reusable shadcn-style components, plus instructions appended to `AGENTS.md` (or `CLAUDE.md`) so future agents always defer to the design system instead of drifting.

## Audience assumption

The user wants a complete, opinionated design system scaffolded into their app with as few decisions as possible. In the usual workflow the design decisions were **already made and client-approved** in a Claude Design mockset — your job is extraction and codification, not design: pull brand color and fonts from the mockset, confirm them, and scaffold. Only when no mockset exists do you fall back to asking for the basics that *must* reflect their brand: brand color, display font, and body font. Everything else (neutral scale, semantic signal/danger colors, spacing, type ramp) ships locked. Whenever a technical concept appears, briefly explain it in plain language before asking.

## Core interaction principles

1. **Use the AskUserQuestion tool for decisions with discrete options.** The user is likely on mobile — tappable options beat typing. For free-form input (custom hex codes, custom font names) use a normal chat message.

2. **One decision at a time, in sequence.** Walk through phases in order. Lock each phase before moving to the next.

3. **Keep your prose tight.** Short framings, no preamble. The user is making decisions, not reading essays.

4. **The design system is opinionated, not exhaustive.** Ship a small, useful, beautiful set of primitives with locked defaults. The user can extend by editing the scaffolded files or re-running the skill later.

## Hard assumptions

- The target codebase uses **React**.
- The target codebase uses **Tailwind CSS v4 or later**. (v4 uses `@theme` directive in CSS instead of `tailwind.config.js`.)
- If either assumption is violated, stop and tell the user what's missing.

## Phase 0 — Detect target codebase state

Before talking to the user, inspect the codebase silently and form a picture.

**Framework detection** (in order — first match wins):

1. **Vite** — `vite.config.ts` / `vite.config.js` / `vite.config.mts` exists at repo root.
2. **Next.js (app router)** — `next.config.*` exists AND `app/` directory exists.
3. **Next.js (pages router)** — `next.config.*` exists AND `pages/` directory exists, no `app/`.
4. **Rails + Inertia** — `Gemfile` contains `inertia-rails` (or `inertia_rails`) AND `app/frontend/` or `app/javascript/pages/` exists.
5. **Rails + react-on-rails** — `Gemfile` contains `react_on_rails`.
6. **Unknown** — none of the above. Continue but tell the user the route registration step will be a manual snippet they paste.

**Tailwind v4 check:**

- Look for `@import "tailwindcss"` in any CSS file under common entry paths (`src/`, `app/`, `app/javascript/`, `app/frontend/`).
- Or `@tailwindcss/vite`, `@tailwindcss/postcss`, or `tailwindcss@^4` in `package.json`.
- If none found, or if you find Tailwind v3 markers (`tailwind.config.js`, `@tailwind base;`), **stop**: print "This skill requires Tailwind CSS v4 or later. Please upgrade first — see https://tailwindcss.com/docs/upgrade-guide — and re-run." and exit.

**Existing-system detection (sets first-run vs re-run mode):**

- Does the route already exist? Look for files matching the framework's convention for `/admin/design-system`.
- Does `src/components/ui/` (or framework-equivalent) already exist?
- Does the project's main CSS file already have a `@theme` block?
- Does the project have a `bm-design-system:start` marker in `AGENTS.md` or `CLAUDE.md`?

If any of those are true → **re-run mode**. Otherwise → **first-run mode**.

**Agent instruction file:**

- Check for `AGENTS.md` at repo root.
- Check for `CLAUDE.md` at repo root.
- Note which exists (or neither).

**Design handoff detection:**

- Check for `_build_plan/design-handoff.md` at the repo root (written by the `/prd-creator` skill).
- If it exists, read it and note whether the "Mockset link" section contains an actual URL (not `TBD` / a placeholder comment).
- This decides the entry into Phase 2: URL present → propose mockset-ingestion mode; file missing or still `TBD` → you'll ask (see Phase 2-pre).

State a brief summary of what you found before moving on. Example:
> Detected: Vite + React + Tailwind v4. No existing design system. `CLAUDE.md` present, `AGENTS.md` absent. Approved mockset link found in `_build_plan/design-handoff.md`. I'll scaffold from scratch, extracting branding from the mockset.

## Phase 1 — Confirm route

Default route: **`/admin/design-system`**.

Use AskUserQuestion:

- Question: "Where should the design system reference page live?"
- Options: `Use /admin/design-system (recommended)`, `Use /design-system (no admin prefix)`, `Use /styleguide`

If the user picks Other and provides a custom path, accept it (must start with `/`).

Save the chosen route as `routePath` for downstream phases.

## Phase 2 — Branding

Resolve three basics — brand (accent) color, display font, body font. Everything else is locked. There are two ways to get them: **Mode A** extracts them from an approved Claude Design mockset (preferred — the client already signed off on those decisions); **Mode B** asks the user directly (fallback when no mockset exists).

### 2-pre. Design source

Decide the mode:

- **Handoff URL detected in Phase 0** → propose Mode A. Use AskUserQuestion:
  - Question: "I found the approved mockset link in `_build_plan/design-handoff.md`. How should I get the branding?"
  - Options: `Extract from the approved mockset (recommended)`, `Pick manually instead`
- **Handoff file missing, or link still TBD** → ask in chat whether an approved Claude Design mockset (or exported mock HTML) exists. If the user provides a link or files, use Mode A with it; otherwise fall through to Mode B.

### Mode A — Extract from the approved mockset

The mockset is the client-approved source of truth; extract the branding rather than re-deciding it.

1. **Fetch the mockset.** Use WebFetch on the handoff URL (and the per-page screens it links to, if needed). If the URL can't be fetched (auth wall, unreachable), ask the user to either paste the mock's exported HTML/CSS or just tell you the three values — don't guess.
2. **Extract:**
   - The **brand accent color** as a 6-digit hex — the color used for primary buttons, links, and highlights in the mocks. If several candidates appear, list them with where each is used and ask the user to pick.
   - The **display font** family (headings) and **body font** family.
3. **Map onto the standard machinery:**
   - Treat the accent hex exactly like a custom hex from Mode B 2a: compute the six accent token values via `references/derive-palette.md`.
   - Treat the fonts exactly like custom font picks from Mode B 2b/2c: resolve stacks and Google Fonts URLs per 2d. If a font isn't available on Google Fonts, say so and ask for a substitute family or a self-hosted stylesheet URL.
4. **Check the neutrals.** The non-accent placeholders (page, surface, hairline, ink) ship locked to the values in 2d. If the mockset clearly deviates — e.g. a dark-only design, tinted surfaces, an unusual page background — surface that in chat and ask whether to override the affected `__COLOR_PAGE_*__` / `__COLOR_SURFACE_*__` / `__COLOR_HAIRLINE_*__` / `__COLOR_INK_*__` values with ones extracted from the mock. Default to the locked values when the mock is a conventional light design.
5. **Confirm before scaffolding.** State the extraction in one line so the user can correct it:

> Extracted from the mockset — **\<headline-font\>** for headings, **\<body-font\>** for body, **\<hex\>** as the brand accent. Scaffolding now.

Then skip Mode B and go straight to **2d** to resolve the substitution values.

### Mode B — Interview (no mockset)

Ask the user for three basics — brand color, display font, body font.

Before the first question, briefly explain in one or two sentences what's about to happen:

> A few quick picks to make this feel like your brand. Each one has good defaults — pick what fits, or pick "Other" to type your own.

### 2a. Brand (accent) color

Use AskUserQuestion:

- Question: "Pick a brand color — this is the splash color used for buttons, links, and highlights."
- Options:
  - `Cyan (default)` — calm, tech-y
  - `Indigo` — modern, software-y
  - `Emerald` — fresh, productive
  - `Rose` — warm, friendly
  - `Violet` — playful, creative
  - `Slate (monochrome)` — restrained, no color splash

If the user picks Other, ask them in chat for a hex code (e.g. `#0891b2`). Validate it's a 6-digit hex (optionally with `#`).

**Resolve the six accent token values:**

- For a curated pick, load the corresponding `references/palettes/<name>.json` (`cyan.json`, `indigo.json`, `emerald.json`, `rose.json`, `violet.json`, `slate.json`) and read `tokens.accent.{light,dark}`, `tokens.accent-faded.{light,dark}`, `tokens.accent-display.{light,dark}`.
- For a custom hex, compute the values via the algorithm in `references/derive-palette.md`. The simplest acceptable behavior:
  - `accent` light = the user's hex; `accent` dark = the user's hex (or a slightly lighter shade if you want polish).
  - `accent-faded` light = user's hex mixed 12% on `#ffffff`; `accent-faded` dark = user's hex mixed 18% on `#020617`.
  - `accent-display` light/dark = the user's hex (acceptable fallback when no darker step is obvious).

### 2b. Display font

Use AskUserQuestion:

- Question: "Pick a display font — used for headings and the page title."
- Options:
  - `Inter` — clean, neutral, very widely used
  - `Plus Jakarta Sans` — modern, friendly
  - `Space Grotesk` — geometric, distinctive
  - `Manrope` — rounded, soft modern
  - `Geist` — Vercel's font, technical feel

If the user picks Other, ask them in chat for a Google Fonts family name (e.g. `Bricolage Grotesque`).

### 2c. Body font

Use AskUserQuestion:

- Question: "Pick a body font — used for paragraphs and most of the UI."
- Options:
  - `DM Sans (default)` — pairs well with most display fonts
  - `Inter` — same family as display if you want one font
  - `Source Sans 3` — workhorse, very readable
  - `Geist` — pair with Geist display
  - `Same as display font` — single-family branding

If the user picks Other, ask them for a Google Fonts family name. If they pick "Same as display font", reuse the family chosen in 2b.

### 2d. Resolve substitution values

After Mode A extraction (or the Mode B picks in 2a–2c), compute these strings for Phase 4 substitution:

- `__HEADLINE_FONT__` = the picked display family name (e.g. `Inter`).
- `__BODY_FONT__` = the picked body family name (e.g. `DM Sans`).
- `__HEADLINE_FONT_STACK__` = `'<family>', ui-sans-serif, system-ui, sans-serif` (quoted family + standard fallbacks).
- `__BODY_FONT_STACK__` = same pattern with the body family.
- `__HEADLINE_FONT_URL__` = `https://fonts.googleapis.com/css2?family=<urlencoded family>:wght@400;500;600;700&display=swap` (display weights).
- `__BODY_FONT_URL__` = `https://fonts.googleapis.com/css2?family=<urlencoded family>:wght@400;500;600&display=swap` (body weights — no 700).
- When display and body are the same family, collapse the two URLs into one: `https://fonts.googleapis.com/css2?family=<family>:wght@400;500;600;700&display=swap` and use that for both `__HEADLINE_FONT_URL__` and `__BODY_FONT_URL__`.
- All twelve `__COLOR_ACCENT_*__` / `__COLOR_ACCENT_FADED_*__` / `__COLOR_ACCENT_DISPLAY_*__` placeholders come from 2a (six values × two modes).
- The remaining color placeholders are **locked** — use these values for every project:

| Placeholder                          | Value                          |
|--------------------------------------|--------------------------------|
| `__COLOR_PAGE_LIGHT__`               | `#ffffff`                      |
| `__COLOR_PAGE_DARK__`                | `#020617`                      |
| `__COLOR_SURFACE_LIGHT__`            | `#f8fafc`                      |
| `__COLOR_SURFACE_DARK__`             | `#0f172a`                      |
| `__COLOR_HAIRLINE_LIGHT__`           | `#e2e8f0`                      |
| `__COLOR_HAIRLINE_DARK__`            | `#1e293b`                      |
| `__COLOR_INK_BODY_LIGHT__`           | `#334155`                      |
| `__COLOR_INK_BODY_DARK__`            | `#e2e8f0`                      |
| `__COLOR_INK_DISPLAY_LIGHT__`        | `#0f172a`                      |
| `__COLOR_INK_DISPLAY_DARK__`         | `#f8fafc`                      |
| `__COLOR_INK_MUTED_LIGHT__`          | `#64748b`                      |
| `__COLOR_INK_MUTED_DARK__`           | `#94a3b8`                      |
| `__COLOR_SIGNAL_LIGHT__`             | `#fcd34d`                      |
| `__COLOR_SIGNAL_DARK__`              | `#fcd34d`                      |
| `__COLOR_SIGNAL_FADED_LIGHT__`       | `#fffaea`                      |
| `__COLOR_SIGNAL_FADED_DARK__`        | `#2f2b21`                      |
| `__COLOR_SIGNAL_DISPLAY_LIGHT__`     | `oklch(76% 0.165 82)`          |
| `__COLOR_SIGNAL_DISPLAY_DARK__`      | `oklch(93% 0.13 95)`           |
| `__COLOR_DANGER_LIGHT__`             | `oklch(64% 0.22 25)`           |
| `__COLOR_DANGER_DARK__`              | `oklch(72% 0.20 22)`           |
| `__COLOR_DANGER_FADED_LIGHT__`       | `oklch(96% 0.025 25)`          |
| `__COLOR_DANGER_FADED_DARK__`        | `oklch(22% 0.05 22)`           |
| `__COLOR_DANGER_DISPLAY_LIGHT__`     | `oklch(55% 0.20 25)`           |
| `__COLOR_DANGER_DISPLAY_DARK__`      | `oklch(82% 0.18 22)`           |

State a one-line confirmation back to the user so they can re-prompt if they don't like a pick (in Mode A you already did this at the end of extraction — don't repeat it):

> Got it — **\<headline-font\>** for headings, **\<body-font\>** for body, **\<color-label\>** as the brand color. Scaffolding now.

One exception to the locked table: if the user approved neutral overrides in Mode A step 4, use those extracted values for the affected placeholders instead.

## Phase 3 — (re-run only) Scope

Skip this phase on first run.

If re-run mode, after Phase 2 ask:

- Question: "What do you want to update?"
- Options: `Refresh tokens to current defaults`, `Add new sections to the page`, `Full re-scaffold (overwrites everything inside the bm-design-system markers)`

If "Add new sections", ask which section IDs to add (free-form, comma-separated against the canonical section list below).

If "Full re-scaffold", confirm with a second AskUserQuestion: `Yes, overwrite`, `Cancel`.

## Phase 4 — Write files

This is the "do the work" phase. Don't show drafts; just write the files. The user has already confirmed the route and the branding (extracted from the mockset or picked manually) in Phase 2.

### 4a. Resolve target paths

Pick the path map for the detected framework. The canonical Vite map is:

| Logical path                           | Vite target                                          |
|----------------------------------------|------------------------------------------------------|
| route page                             | `src/admin/design-system/page.tsx`                   |
| design-system components               | `src/components/design-system/*.tsx`                 |
| ui primitives                          | `src/components/ui/{button,input,label,dialog}.tsx`  |
| utils                                  | `src/lib/utils.ts`                                   |
| design-system stylesheet               | `src/styles/design-system.css`                       |
| entry CSS (where to add the import)    | the file containing `@import "tailwindcss"`          |

Per-framework overrides:

- **Next.js app router** — route page at `app/admin/design-system/page.tsx`; everything else at `components/...`, `lib/...`, `styles/...` (no `src/` prefix unless `src/` exists).
- **Next.js pages router** — route page at `pages/admin/design-system.tsx`.
- **Rails + Inertia** — route page at `app/frontend/pages/admin/design-system.tsx` (or `app/javascript/pages/...` if that's the pattern); update Inertia routes file accordingly.
- **Rails + react-on-rails** — print a manual snippet pointing to the user's existing component registration; do not auto-edit Ruby files.
- **Unknown** — write components into `src/components/...` if `src/` exists, otherwise the project root, and print a manual route registration snippet.

### 4b. Copy the templates

(See also Phase 4e for the theme boot script.)


For each template under `references/`, write to its mapped target. Substitute these tokens (literal string-replace) at write time. **Substitution applies to `.css`, `.ts`, `.tsx`, and `.html` template files** — Phase 4b will substitute into `design-system.css`, `palette.ts`, `TypographySection.tsx`, and `font-link-tags.html`.

| Token in template                  | Replacement                                                                              |
|------------------------------------|------------------------------------------------------------------------------------------|
| `__ROUTE_PATH__`                   | the chosen `routePath`                                                                   |
| `__HEADLINE_FONT__`                | the headline font family name (bare, no quotes — e.g. `Inter`)                           |
| `__BODY_FONT__`                    | the body font family name (bare, no quotes — e.g. `DM Sans`)                             |
| `__HEADLINE_FONT_STACK__`          | full CSS stack — `'<family>', ui-sans-serif, system-ui, sans-serif`                      |
| `__BODY_FONT_STACK__`              | full CSS stack — same pattern with the body family                                       |
| `__HEADLINE_FONT_URL__`            | Google Fonts URL for the headline font (display weights 400–700)                         |
| `__BODY_FONT_URL__`                | Google Fonts URL for the body font (body weights 400–600)                                |
| `__COLOR_<TOKEN>_<MODE>__`         | resolved color value — hex (e.g. `#0f172a`) or `oklch(...)` for the locked semantic colors. See the full table in Phase 2d. |

`<TOKEN>` and `<MODE>` combinations used in the templates: `PAGE`, `SURFACE`, `HAIRLINE`, `INK_BODY`, `INK_DISPLAY`, `INK_MUTED`, `ACCENT`, `ACCENT_FADED`, `ACCENT_DISPLAY`, `SIGNAL`, `SIGNAL_FADED`, `SIGNAL_DISPLAY`, `DANGER`, `DANGER_FADED`, `DANGER_DISPLAY` × `LIGHT` / `DARK` — 30 placeholders total.

Files to write (sources under `references/`, plus their substitution behavior):

- `page/DesignSystem.tsx` → `components/design-system/DesignSystem.tsx` (Next.js app-router clients also need a `"use client"` directive)
- `page/{SidebarNav,SectionShell,CodeBlock,ColorSwatch}.tsx` → `components/design-system/`
- `page/palette.ts` → `components/design-system/palette.ts` — **substitute color + font tokens** here
- `page/sections/**/*.tsx` → `components/design-system/sections/`. Note: all 14 base-styles sub-sections live in a single `BaseStylesSection.tsx` (returns a fragment of 14 anchored `SectionShell`s); the canonical section list maps to anchors, not files.
- `components-ui/{button,button-dropdown,input,badge,dialog,checkbox,radio,select,rich-text-field,dropdown-menu,theme-toggle}.tsx` → `components/ui/` (skip a file if it already exists and the existing one already comes from this skill — check for a `bm-design-system` marker comment; otherwise ask the user before overwriting). Note: `rich-text-field.tsx` imports `@milkdown/crepe` and `dropdown-menu.tsx` imports `@radix-ui/react-dropdown-menu` — make sure the deps in 4f are installed before the user navigates to those sections, or the page will fail to render.
- `styles/design-system.css` → target stylesheet path; **substitute color + font tokens** here
- `lib/utils.ts` → `lib/utils.ts` only if missing
- `lib/theme.ts` → `lib/theme.ts` (the `useTheme` hook + helpers backing `<ThemeToggle>`)
- A small route-page entry file at the framework's location (e.g. `src/admin/design-system/page.tsx` for Vite) that imports and renders `DesignSystem`

### 4c. Wire the stylesheet

Append `@import "./design-system.css";` (or correct relative path) to the project's existing entry CSS, immediately after `@import "tailwindcss";`. If the import is already present, skip. Wrap the import in `bm-design-system:start` / `bm-design-system:end` HTML-comment-style CSS block markers so re-runs can be non-destructive:

```css
/* bm-design-system:start */
@import "./design-system.css";
/* bm-design-system:end */
```

**Tailwind v4 source detection — Rails + Inertia and other split-source projects.** Tailwind v4 normally auto-detects which files contain class names, but the auto-detection doesn't reliably cover projects where the source roots are split across multiple top-level directories — most notably **Rails + Inertia**, which has page components under `app/javascript/pages/` and shared components under `app/frontend/`. If the entry CSS lives in `app/javascript/entrypoints/` (or anywhere that doesn't naturally walk up to both roots), classes used in the other root will be silently dropped from the build and pages will render unstyled.

For Rails + Inertia (and any other framework where the source roots are split), add an explicit `@source` directive to the entry CSS that walks up to the closest common ancestor of all source roots. For Rails + Inertia with the entry CSS at `app/javascript/entrypoints/application.css`, that's:

```css
@import "tailwindcss";

/* Tailwind v4 auto-detection doesn't reliably reach both `app/javascript/`
   and `app/frontend/` in this Rails+Inertia setup, so declare them explicitly. */
@source "../../**/*.{ts,tsx,js,jsx}";

/* bm-design-system:start */
@import "./design-system.css";
/* bm-design-system:end */
```

Place the `@source` line *between* the `tailwindcss` import and the design-system import. For other frameworks (Vite with everything under `src/`, Next.js with everything under `app/`), Tailwind's auto-detection works and an explicit `@source` is unnecessary — skip it.

### 4d. Register the route

Use the corresponding snippet under `references/routing/`. Edit the user's router file in place when the framework supports clean detection (Vite + react-router-dom: look for the `<Routes>` block; Next.js: file-based, no edit needed). Otherwise print the snippet and tell the user where to paste it.

### 4e. Install the HTML layout head snippets

Two snippets need to land inside the app's HTML layout `<head>`, before any `<script>` that loads React / hydration code (Vite tags, Inertia entrypoint, Next.js scripts, etc.):

#### 4e-i. Font `<link>` tags

The `@theme` block defines `--font-display` and `--font-sans` using the families the user picked in Phase 2, but those CSS variables only resolve to the right typeface if the font files are actually loaded. Don't load fonts via `@import url(...)` inside `design-system.css` — bundlers inline that file mid-bundle, after Tailwind's rules. CSS spec says `@import` must come before any other rules, so browsers silently drop those imports and the page renders in the system fallback (`ui-sans-serif`).

Install the link-tag snippet from `references/styles/font-link-tags.html` into the layout's `<head>`. Substitute `__HEADLINE_FONT_URL__` and `__BODY_FONT_URL__` with the URLs picked in Phase 2. When both fonts are Google Fonts, prefer collapsing them into a single `&family=…` stylesheet for one fewer request.

#### 4e-ii. Theme boot script

The `<ThemeToggle>` primitive persists the user's choice to `localStorage["bm-ds-theme"]`, but the React hook that reads it (`useTheme` in `lib/theme.ts`) only fires when a component using it actually mounts. In real apps the toggle commonly lives inside a portal-rendered dropdown that doesn't mount until the user opens it — so without a boot-time script, every full page load renders in light mode regardless of the saved preference.

Install the inline script from `references/styles/theme-boot-script.html` into the same `<head>`. It reads the same storage key and resolution rules as `lib/theme.ts`, then adds (or omits) the `.dark` class on `<html>` synchronously before paint.

#### Per-framework target for both snippets

- **Rails + Inertia** — `app/views/layouts/application.html.erb`, immediately above `vite_javascript_tag` / `vite_typescript_tag`.
- **Next.js (app router)** — inside `app/layout.tsx`'s `<head>` (font links via `next/font` or plain `<link>` tags; theme boot via `<Script id="theme-boot" strategy="beforeInteractive">{...}</Script>` or a `dangerouslySetInnerHTML` script).
- **Next.js (pages router)** — `pages/_document.tsx`, inside `<Head>` of the custom `Document`.
- **Vite + react-router-dom** — `index.html`, inside `<head>` before `<script type="module" src="/src/main.tsx"></script>`.

Wrap each inserted block in `bm-design-system:start` / `bm-design-system:end` HTML comment markers so re-runs can replace them idempotently.

### 4f. Report missing dependencies

Inspect `package.json`. For any of the following that are missing, append them to a single install command and print it (do **not** run npm/yarn yourself):

- `@radix-ui/react-dialog`
- `@radix-ui/react-dropdown-menu`
- `class-variance-authority`
- `clsx`
- `tailwind-merge`
- `lucide-react`
- `@milkdown/crepe`
- `@milkdown/core`
- `@milkdown/react`

Example:
> Run this to install missing deps:
> `npm install @radix-ui/react-dialog @radix-ui/react-dropdown-menu class-variance-authority clsx tailwind-merge lucide-react @milkdown/crepe @milkdown/core @milkdown/react`

## Phase 5 — Update agent instructions

Open `AGENTS.md` if it exists, else `CLAUDE.md` if it exists, else create `AGENTS.md`. If both exist, update both the same way so they stay in sync.

### 5a. Reconcile existing UI / styling directives

Before appending the managed block, read the file end-to-end and look for any **pre-existing instructions about UI, CSS, design, styling, color, typography, components, or frontend conventions** that live *outside* a `bm-design-system:start`/`end` block. Examples of what to flag:

- "Use Tailwind classes like `bg-blue-600` for primary buttons"
- "Headings should use Inter / our custom font stack"
- A "Styling" / "Design" / "UI conventions" / "Components" / "Frontend" section
- Per-component rules ("buttons should be rounded-md", "cards use `bg-gray-50`")
- Color palette references, hex codes, font names, spacing scales
- Pointers to other style guides or component libraries

These will conflict with — or quietly override — the new design system if left in place. Resolve every conflict found:

- **If the existing directive is fully superseded** (e.g. a hardcoded color palette, a "use these utilities" list, a font directive), **remove it**.
- **If the existing directive carries non-styling intent worth keeping** (e.g. "all forms must be accessible", "preserve the existing route structure"), **rewrite it** to drop the styling specifics and keep the intent, and add a deferral clause like "follow the design system at `__ROUTE_PATH__` for visual conventions."
- **If you're not sure** whether something should go, surface it to the user in chat — quote the line, say what you'd do, and ask before editing. Default to keeping content when ambiguous.

Make the edits in place, briefly note in chat what you removed or rewrote, then proceed to 5b.

### 5b. Append the managed block

Append (or replace, if the markers already exist) the block from `references/agent-instructions.md`. The block is delimited by HTML comments:

```markdown
<!-- bm-design-system:start -->
…content from references/agent-instructions.md, with __ROUTE_PATH__ substituted…
<!-- bm-design-system:end -->
```

If the file is being created from scratch, also include a one-line top-level title above the block.

## Phase 6 — Scan for existing UI to migrate

After the scaffold is in place, do a quick scan for **user-facing UI that already exists in the codebase** and was not written using the design system. The point is to surface the migration opportunity — not to do it now.

### 6a. Detect

Scan the framework's standard UI locations (skip the route page you just wrote, and skip everything under `components/design-system/`, `components/ui/`, and `lib/`):

- **Vite** — `src/pages/**`, `src/routes/**`, `src/views/**`, `src/components/**` (excluding `src/components/ui/` and `src/components/design-system/`)
- **Next.js app router** — `app/**/page.tsx`, `app/**/layout.tsx`, `app/**/*.tsx` (excluding `app/admin/design-system/**`), and `components/**` (excluding `components/ui/` and `components/design-system/`)
- **Next.js pages router** — `pages/**/*.tsx` (excluding `pages/admin/design-system.tsx`), and `components/**`
- **Rails + Inertia** — `app/frontend/pages/**` or `app/javascript/pages/**` (excluding the design-system page), plus the components directory
- **Rails + react-on-rails** — same component directories the user has registered

For each candidate file, look for **signals that it renders user-facing UI without the design system**:

- Raw `<button>` / `<a>` / `<input>` / `<select>` / `<form>` elements with their own ad-hoc Tailwind classes (instead of `<Button>`, `<Input>`, `<Select>`, etc.)
- Inline color utilities that bypass tokens — `bg-white`, `bg-gray-50`, `bg-slate-100`, `text-gray-900`, `text-zinc-500`, `border-gray-200`, raw hex via `style={{...}}`, etc.
- Page shells / layouts / headers / footers built directly in JSX rather than reusing the structure sections
- Nav menus, sidebars, and cards built ad-hoc

Group findings into broad buckets (don't list every file individually — just pattern + count + one or two example paths):

- Pages / routes
- Layouts / shells / headers / footers
- Navigation / menus
- Forms / inputs
- Buttons / links
- Cards / listings / content blocks

If the scan finds **nothing meaningful** (e.g. brand-new project, or everything already follows the system), say so in one sentence and skip 6b. Move on to Phase 7.

### 6b. Offer migration

If there's UI to migrate, summarize what you found in chat — short, scannable. Then ask, using AskUserQuestion:

- Question: "Want me to migrate the existing UI to use the new design system?"
- Options:
  - `Yes, migrate everything now` — agent walks the codebase file-by-file, replacing ad-hoc styles and elements with tokens and primitives. Confirm with the user before each substantial file change.
  - `Yes, but just <bucket>` — narrow scope (e.g. only buttons, only pages). Ask which bucket if they pick this.
  - `Not now` — skip; user can re-run later.

If the user picks a migration option, proceed. Otherwise move on. Do not auto-migrate without explicit user opt-in — design-system migrations touch a lot of code and need explicit consent.

### 6c. Migration guidance (if user opted in)

When migrating:

- Replace raw `<button>` with `<Button>` and pick the closest variant; same for `<input>`, `<select>`, `<a>` styled-as-button, `<form>` field wrappers.
- Replace inline color utilities with tokens — `bg-white` → `bg-page`, `bg-gray-50` → `bg-surface`, `text-gray-900` → `text-ink-display`, `text-gray-500` → `text-ink-muted`, `border-gray-200` → `border-hairline`, `bg-blue-600` → `bg-accent`, etc. When the original color clearly isn't semantic (e.g. a brand-specific color used once), flag it in chat and ask before substituting.
- Wrap long-form prose blocks in `.body-content`.
- Keep the user in the loop: report progress per file or per bucket, surface anything ambiguous, and stop to ask rather than guessing.
- Do not change behavior or copy — only styles, structure, and primitive substitution.

## Phase 7 — Wrap up

Print a short summary in chat:

- The route URL the user can visit
- How to toggle dark mode (button in the header — also persists in localStorage)
- Which agent file got the managed block
- The install command (if dependencies were missing)
- What was migrated in Phase 6 (or "no migration needed" / "user deferred migration")
- A nudge: "Re-run this skill any time to add new sections, refresh tokens, or re-scan for migration candidates — it detects existing setup and merges non-destructively."

Stop.

## Canonical section list

The page contains these sections, in this order, each with an anchor ID matching the slug:

```
branding/
  colors                #colors
  typography            #typography
structure/
  shells                #shells
  main-navigation       #main-navigation
  sub-navigation        #sub-navigation
  page-headers          #page-headers
  body-content          #body-content
  footers               #footers
elements/
  iconography           #iconography
  buttons               #buttons
  button-dropdown       #button-dropdown
  forms                 #forms
  badges                #badges
  toggle-buttons        #toggle-buttons
  listings              #listings
  modal                 #modal
  dropdown-menu         #dropdown-menu
  callout               #callout
base-styles/
  heading-scale         #heading-scale
  h1                    #h1
  h2                    #h2
  h3                    #h3
  h4                    #h4
  h5                    #h5
  h6                    #h6
  anchor                #anchor
  paragraph             #paragraph
  strong                #strong
  lists                 #lists
  list-item             #list-item
  blockquote            #blockquote
  hr                    #hr
  label                 #label
```

Each section uses the `<SectionShell>` wrapper which renders five blocks in this order:

1. **Preview** — the live, fully-styled element(s)
2. **Description** — one short paragraph
3. **Use cases** — bulleted: when to use, when not to use
4. **Sample HTML** — copy-pasteable JSX/HTML
5. **Options / variations** — for components with variants (Button sizes/styles, Modal sizes, etc.); omitted for primitives like `<hr>`

## Token derivation reference

See `references/derive-palette.md` for the exact algorithm that turns the locked accent / signal / neutral defaults into the full ten-token light + dark palette.

## Style notes for the scaffolded page

- The page itself uses the design system it documents (eat your own dog food).
- Sidebar is sticky on desktop, collapses to a top sheet (button-triggered) on mobile.
- Each section has a unique `id` matching the canonical slug above for hash-link smooth scrolling.
- All section anchors use `scroll-margin-top` to clear the sticky header.
- Dark mode is class-based on `<html>` (Tailwind v4 `@variant dark` against `.dark`), persisted to `localStorage["bm-ds-theme"]`.
- The page is the **reference**, not the **boundary** — the user can extend it. The reusable primitives in `components/ui/` are the actual API the rest of the app imports.
