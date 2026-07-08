# Write files

Once everything is locked, generate the files. **Just write them.** Don't show a draft for approval first — the user already approved each piece during the interview.

## What to write

Always write **both** PRD files — they serve different audiences from the same locked scope:

- `_build_plan/prd.md` — the **full PRD**, the source of truth. The builder and the milestone coding agents consume this one. Use the markdown template below, complete with the build-plan note and milestone-log mechanics.
- `_build_plan/prd.html` — the **client-friendly PRD** for review and sign-off. Use the scaffold and section snippets in `steps/prd-html-template.md`. Same locked scope and voice, but omit builder-internal mechanics: the "About these build-plan files" agent note, milestone-log instructions, and anything else that only matters to the coding agent.

Also write:

- `_build_plan/design-brief.md` — a paste-ready brief for Claude Design (template below), built from the locked scope plus the **Design inputs** phase.
- `_build_plan/design-handoff.md` — a stub (template below) where the builder records the approved mockset link after client sign-off. Milestone prompts reference it.
- `_build_plan/milestones/{N}-{slug}/prompt.md` — one per milestone, always markdown; consumed by the coding agent in plan mode, not by the user.

Create this exact structure in the codebase root:

```
_build_plan/
  prd.md               # full PRD — builder + coding agents
  prd.html             # client-friendly PRD — review & sign-off
  design-brief.md      # paste into Claude Design to generate the mockset
  design-handoff.md    # stub — approved mockset link goes here after sign-off
  milestones/
    1-{milestone-slug}/
      prompt.md
    2-{milestone-slug}/
      prompt.md
    ...
```

`{milestone-slug}` is a short kebab-case name derived from the milestone (e.g., `core-crud`, `integrations-layer`, `public-docs`).

After writing the `_build_plan/` files, also add a short note about the `_build_plan/` folder to the project's agent instructions file — see "Agent instructions note" below.

After writing, briefly tell the user the files are ready and how to use them, in workflow order:

1. Open `_build_plan/prd.html` in a browser (`open _build_plan/prd.html`) and send it to the client for PRD review; `prd.md` is the full version for you and the coding agents.
2. Once the PRD is approved, paste `_build_plan/design-brief.md` into [Claude Design](https://claude.ai/design) (along with the client's design system, if they have one there) and iterate on the mockset with the client.
3. When the client approves the mockset, paste its handoff/share URL into `_build_plan/design-handoff.md` — the milestone prompts read it from there.
4. Then hand the agent the milestone-1 `prompt.md` to start building. Mention that after each milestone the agent writes a `milestone-log.md` in that milestone's folder to record what was done.

## File templates

### prd.html structure

See `steps/prd-html-template.md` for the complete scaffold, per-section snippets, and icon hints. Read that file in full when generating `prd.html` and follow it closely. Remember the audience: this is the client's review copy — same locked scope as `prd.md`, but leave out the agent-facing build-plan note and the milestone-log mechanics.

### prd.md structure

Mirror the structure of a high-quality real PRD. Use these sections in order:

```markdown
# {App name}

> **About these build-plan files:** Everything in `_build_plan/` (this PRD and the per-milestone folders) is a **temporary documentation and guidance artifact** for the initial build-out of this codebase. These files are not functional — no code, configuration, runtime logic, tests, or deployment process should import, read, reference, or depend on anything in `_build_plan/`. Once the initial milestones are built and shipped, the entire `_build_plan/` folder is expected to be deleted from the codebase. Do not treat it as long-living documentation.

## What we're building

{1–3 sentence core purpose, expanded with a paragraph or two of context. End with a sentence on the tech stack and how the build is structured around milestones.}

---

### What the app does

{Bulleted list of the high-level user-facing capabilities, written from the user's perspective. 5–10 bullets.}

---

### Already provided by the {starter template name, or "existing codebase"}

{Bulleted list of what's already built and does not need to be re-specced.}

---

### Out of scope

{Top-level out-of-scope list with brief reasoning for each item. Each bullet is one line.}

---

### Data model

{For each entity, a heading and a bullet list of fields described in plain language — what the app needs to remember about this thing, not the database column types or constraints. Note relationships in prose between entities or at the end. Keep this conceptual, not technical: "url — the link being saved", not "url: string, not null, indexed."}

---

## Milestone 1 — {Name}

{1–2 sentence framing of what this milestone delivers.}

### What gets built

{Bulleted list of user-facing capabilities and screens delivered in this milestone. Describe what the user can do, see, or experience when this milestone is done — not the technical pieces (controllers, models, jobs) needed to deliver it. The agent will figure out the technical pieces in plan mode.}

### What milestone {N} explicitly does NOT include

{Bulleted list of things a coder might assume should be in this milestone but aren't.}

### Done when

{1–2 sentences describing the verification criteria — what the user should be able to do in the browser when this milestone is complete.}

---

{Repeat for each milestone}
```

### milestones/N-{slug}/prompt.md structure

Keep this lean. The prompt.md is a thin trigger file — it does NOT re-summarize what's in the PRD.

```markdown
# Milestone {N} — {Name}

You are entering plan mode to plan and then build milestone {N} of this project.

## Context

- Read `@_build_plan/prd.md` for the full project context, scope, data model, and tech stack.
- Read `@_build_plan/design-handoff.md` for the approved Claude Design mockset link. Open the mockset screen(s) for this milestone's features and match their layout, components, and visual hierarchy — implemented with the app's design-system tokens and `components/ui/` primitives (see the `/admin/design-system` reference page), not by copying ad-hoc styles out of the mock. Where the mock and the design system conflict, use the design-system tokens and flag the difference. If the handoff file has no URL, ask the user for it before building.
- Read previous milestone folders (`@_build_plan/milestones/1-*/milestone-log.md`, etc.) to understand what has already been built. If you are working on milestone 1, there is no prior milestone to read.

## Your task

1. Plan the implementation for **only** milestone {N} as defined in the PRD. Do not plan or build anything from later milestones.
2. After the user confirms the plan, build only what is in milestone {N}'s scope.
3. Verify your work against the "Done when" criteria for milestone {N} in the PRD.
4. When complete, write a `milestone-log.md` in this folder (`_build_plan/milestones/{N}-{slug}/milestone-log.md`). Structure it as follows:
   - **Start with a `## What's new in the app` section at the very top.** This is a concise, human-readable, bulleted list of the main user-facing features or functionality that were added in this milestone — written so a non-technical reviewer can see at a glance what new things to expect in the app now that this milestone is done. Frame each bullet as a capability the user will now see or be able to do, not as a technical artifact. Keep it short and scannable.
   - Then include the implementation detail sections below for the next milestone's agent to reference:
     - What was built (files created, models added, routes added, etc.)
     - Any decisions made during implementation that weren't pre-specified in the PRD
     - Anything the next milestone will need to know
     - Any deviations from the PRD and why

Ask me any clarifying questions using AskUserQuestion tool to lock in the implementation plan for this milestone.
```

### design-brief.md structure

A self-contained packet the builder pastes into Claude Design to generate the mockset. It must stand alone — Claude Design won't see the PRD or this repo. Keep it in the same plain, non-technical voice as the PRD: what each screen shows and does, never how it's coded.

```markdown
# Design brief — {App name}

You are designing a high-fidelity, interactive mockset for a web application. Produce one screen per key screen listed below, as an integrated set (shared navigation/shell, consistent components), responsive web layout.

## What the app is

{The locked core purpose, 1–3 sentences, plus one line of business context.}

## Design system & brand

{The Design inputs captured earlier: the client's Claude Design design system or brand guidelines to follow, brand colors/fonts/logo, reference apps they like or want to avoid, light/dark and feel preferences. Mark anything not yet known as **TBD — fill in before pasting this brief**.}

## Screens to design

{One subsection per key screen, derived from the locked features and per-feature scoping, in the order users encounter them. For each: the screen name, the job it does for the user, what's on it (content, primary actions, key states like empty/loading), and how the user gets to and from it. Condensed from the per-feature specs — enough for a designer to lay out the screen, no implementation detail. Skip screens the starter already provides (login/signup, settings) unless the PRD changes them.}

## Out of scope

{The locked out-of-scope list, so the mocks don't invent features that were cut.}
```

### design-handoff.md structure

A stub the builder fills in after the client approves the mockset. Milestone prompts read it, so write it exactly like this:

```markdown
# Design handoff

## Mockset link

<!-- Paste the approved Claude Design handoff/share URL here after client sign-off. -->

TBD

## Per-screen notes (optional)

<!-- Anything the mockset doesn't capture: agreed deviations, screen-specific guidance, components to reuse across screens. -->
```

## Agent instructions note

After writing the `_build_plan/` files, append a short note to the project's agent instructions file so future agent sessions understand the role of `_build_plan/`.

1. Check the codebase root for an existing `CLAUDE.md` or `AGENTS.md`. Use whichever exists.
2. If neither exists, create `AGENTS.md` at the codebase root.
3. Append the section below at the **bottom** of the file (after any existing content). If a `## _build_plan/` section already exists, update it in place rather than duplicating.

```markdown
## `_build_plan/`

The `_build_plan/` folder contains the initial PRD, the design brief/handoff, and per-milestone prompts used to scaffold this codebase during its initial build-out phase. These files are **temporary** — they exist for documentation and guidance only. They are **not** functional: no code, configuration, or runtime logic in this codebase should import, reference, or depend on anything inside `_build_plan/`.

Do not treat `_build_plan/` as long-living documentation for the codebase. The codebase will evolve past the assumptions and decisions captured here. Once the initial milestones are complete, this folder is expected to be deleted.
```

## Style notes for the PRD output

- Mirror the voice of a sharp product spec: concrete, specific, opinionated. Not "the app should probably support X" but "the app supports X."
- Per-feature scoping is specific about user-facing behavior: what the user sees on screen, what they can do, what they cannot do, what the output looks like. It is NOT specific about technical implementation (timeouts, libraries, error-handling patterns, parsing logic) — that's the agent's job in plan mode.
- The "Out of scope" lists are valuable — never skip them, never make them generic.
- Data model fields are described in plain language (what the app needs to remember), not as database column definitions.
- When referring to the starter template features, use the actual names if known (e.g., "Build New starter" rather than "the starter template").
- These style notes apply to both PRD files. The HTML version uses the same locked scope as the markdown — a different presentation (minus the builder-internal mechanics), not a different plan.
