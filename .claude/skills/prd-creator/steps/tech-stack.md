# Tech stack & starter template

**First, detect what's already there.** Without asking the user, check the codebase:

1. Read `CLAUDE.md` and/or `AGENTS.md` if either exists — these often spell out the stack and conventions.
2. Confirm the starter signature: this template is a **Next.js (App Router) + Supabase** starter — `next` in `package.json` with an `app/` directory, Supabase browser/server clients under `lib/supabase/` with the auth session cookie proxy in `proxy.ts`, shadcn/ui primitives under `components/ui/`, and Tailwind CSS v4 (CSS-first config in `app/globals.css`; there is no `tailwind.config.ts`).
3. If the codebase has grown past the starter (existing features, tables, routes beyond the home page), note what's already built so the PRD doesn't re-spec it.

**Then summarize what you found** to the user in plain language: "You're in the Next.js + Supabase starter. That gives you the app shell, the styling system, dark mode, Supabase connectivity, and one-command Vercel deploys out of the box — the PRD doesn't need to spec any of that."

**Lock what the starter already provides** so the PRD doesn't re-spec it:

- Next.js App Router + TypeScript, deployed to Vercel via the `deploy-to-vercel` skill (previews on every push, promote to production on ask)
- Tailwind CSS v4 + shadcn/ui primitives + dark mode (next-themes) — the full design system (tokens, components, live reference page) is scaffolded from the client-approved mockset by the `/design-system` skill
- Supabase wired up as the backend: browser + server clients (`lib/supabase/`), the auth session cookie proxy, and env-var configuration (`.env.local`)

**What the starter deliberately does NOT provide: auth screens.** There are no login, signup, or password pages in the template. The PRD must decide the auth approach — SSO / social login via Supabase Auth, email/password, magic links, or none for a public app — and the auth screens and flows are specced like any other feature and built in a milestone. Only include auth screens in the Claude Design mockset if their look matters to the client; otherwise standard patterns are fine.

Confirm the auth approach with AskUserQuestion:

- **SSO / social login (recommended for client work)** — users sign in with Google/Microsoft/etc. via Supabase Auth; no passwords to manage
- **Email + password** — classic signup/login/reset flows via Supabase Auth
- **No auth in v1** — public app, or auth deferred to a later milestone

**If detection shows a different stack** (this isn't the starter, or the user says so), summarize what you actually found, confirm it with the user in chat, and lock that instead — don't force the starter assumptions onto a different codebase.
