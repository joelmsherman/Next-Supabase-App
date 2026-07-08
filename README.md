# Next.js / Supabase Starter

## Introduction

A template for greenfield Next.js + Supabase products, pre-wired for **agent-first development**.

## Prerequisites

### 1. Node.js v22+
npm is included with Node.js — no separate install needed. Next.js 16 requires Node 20.9+, and the Supabase client libraries require Node **22 or higher**.

- **Check your installed version:**
```bash
node -v
```
If you see `v22.x` or higher, you're good. If not, install or update Node.js:

- **macOS/Linux (via nvm):**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
nvm install 22
nvm use 22
```

- **Windows:** Download the installer from [nodejs.org](https://nodejs.org/)

### 2. Supabase Project
A Supabase account, organization and project is required.

- [Signup](https://supabase.com/dashboard/signup).
- Create a [new project](https://database.new).

### 3. Vercel Account
A Vercel account and the Vercel CLI is required.

- [Signup](https://vercel.com/signup). Use the **same GitHub account** that will host your repo so auto CI works
- Install the Vercel CLI globally and authenticate
```bash
npm install -g vercel
vercel login
```

## Setup

1. Clone this template to a local repository `repo-name` of your choice:
```bash
git clone https://github.com/joelmsherman/Next-Supabase-App.git /path/to/your/<repo-name>
cd /path/to/your/<repo-name>
```

2. Install dependencies:
```bash
npm install
```

3. Disconnect from the template's remote and create an empty one on GitHub (no README/license/.gitignore)
```bash
git remote remove origin
```

4. Point your local at your new GitHub remote and push
```bash
git remote add origin https://github.com/<your-user>/<your-repo>.git
git branch -M main
git push -u origin main
```

5. Configure Supabase
In your Supabase project dashboard, click **Connect** --> **Framework**, set `Framework` = Next.js, `Variant` = App Router; copy the env-var block, paste it over `.env.example` and rename to `.env.local`.

6. Configure Supabase Skills and MCP Server access
In your Supabase project, click **Connect** --> **MCP**, set `Client` = Claude Code, select all feature groups, click **Copy Prompt**, and deliver it to Claude Code. Claude will install Supabase Agent Skills and register the project-scoped MCP Server.

7. Configure Vercel Skills and MCP Server access
```bash
`npx skills add https://github.com/vercel-labs/agent-skills --skill deploy-to-vercel`
```
```bash
mcp add --transport http vercel https://mcp.vercel.com`
```

8. Verify
```bash
npm run dev
```
Home page should display "Can't wait to see what you build"

## Workflow

### Step 1 - Create PRD
In Claude Code, invoke the prd-creator skill, and pass along any other context about the product you might have (intake meeting transcript, notes, etc.). Claude interviews you about the product, then writes everything to `_build_plan/`:

- `prd.md` — the full PRD, consumed by you and the milestone coding agents
- `prd.html` — the client-friendly PRD; open it in a browser and send it to the client for review
- `design-brief.md` — a paste-ready brief for Claude Design (next step)
- `design-handoff.md` — a stub where the approved mockset link goes after sign-off
- `milestones/N-{slug}/prompt.md` — one build prompt per milestone

```
/prd-creator
```

### Step 2 - Design mocks
Once the client approves the PRD, paste `_build_plan/design-brief.md` into [Claude Design](https://claude.ai/design) — along with the client's design system, if they have one there — and create high-fidelity interactive mocks. Iterate with the client until final approval.

When approved, **paste the mockset's handoff/share URL into `_build_plan/design-handoff.md`**. The `/design-system` skill (step 3) and every milestone prompt (step 4) read it from there.

### Step 3 - Setup Design System
In Claude Code, invoke the design-system skill. Claude detects the approved mockset link in `_build_plan/design-handoff.md`, extracts the brand color and fonts from the mockset, confirms them with you, then scaffolds the full design system, including a live reference page at `/admin/design-system` that future agents are instructed to defer to. If there's no mockset, it falls back to three quick picks — brand color, display font, body font (curated options with an "Other" fallback for custom hex / Google Fonts).
```
/design-system
```

### Step 4 - Implement each Milestone
Hand each milestone prompt back to Claude **in Plan Mode** — the prompt already points at the PRD and the approved mockset via `_build_plan/design-handoff.md`. Answer clarifying questions, review the plan, approve, test (`npm run dev`), then [deploy](#deployment). Move to the next milestone, rinse and repeat.

## Deployment

Deployment is driven by the `deploy-to-vercel` skill installed in [Getting Started → step 7](#getting-started). The skill detects whether your project is linked to Vercel, links it on first run using the GitHub remote you set up in steps 3–4, deploys, and returns the URL. **Previews are the default**; production requires an explicit ask. Here are the Vercel deployment workflow steps:

1. First deploy (preview) Prompt for Claude:

> Deploy this app to Vercel as a preview and give me the link.

Claude will create and link a new Vercel project (scoped to the team you choose), deploy a preview, and return the URL.

2. Inject Supabase env vars into the new Vercel project. Prompt Claude:

> Add `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` from my `.env.local` to the Vercel project for Production, Preview, and Development.

Claude will use the Vercel MCP server to set both env vars across all three environments. Alternatively, you can add them by hand in the Vercel dashboard under **Project → Settings → Environment Variables**. Trigger a redeploy after the vars are set so the build picks them up.

3. For ongoing deployments, every push to a non-`main` branch or PR **auto-deploys a preview**. Just ask Claude to commit and push when you're ready for a preview. **For a production deploy**, merge your PR into `main`, or prompt Claude:

> Promote the latest preview to production.

4. **After your first production deployment**, tell Claude to allow-list the new domains for Supabase auth redirects:

> Add my Vercel production domain and the `*.vercel.app` preview wildcard to the Supabase auth redirect URL allow-list.

The Supabase MCP installed in step 3 lets Claude configure this directly.

## Folder Structure

```
├── app/                 # Next.js App Router
│   ├── globals.css      # Tailwind v4 CSS-first config + shadcn tokens
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ui/              # shadcn/ui primitives
│   └── theme-switcher.tsx
├── lib/
│   ├── supabase/        # browser + server Supabase clients
│   └── utils.ts
├── _build_plan/         # written by /prd-creator: PRDs (full + client), design brief/handoff,
│   └── milestones/      #   and per-milestone prompts — temporary; delete after build-out
├── proxy.ts             # Supabase auth session cookie proxy
├── components.json      # shadcn/ui config
├── postcss.config.mjs   # Tailwind v4 PostCSS plugin (no tailwind.config.ts)
├── next.config.ts
├── tsconfig.json
├── .env.example
└── package.json
```

The starter ships intentionally lean — **no auth screens or demo pages**. The auth approach (SSO / social login or email + password via Supabase Auth) is decided during the PRD interview, and the auth screens are specced and built like any other feature during milestones. The Supabase plumbing (browser/server clients, session cookie proxy) is already wired up and waiting.