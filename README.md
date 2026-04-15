# Next.js / Supabase Starter App Template

## Introduction

A Next.js + Supabase starter template optimized for **agent-first development**. It bundles the [Supabase Next.js quickstart](https://supabase.com/docs/guides/getting-started/quickstarts/nextjs) with the Supabase and Vercel MCP servers, the Vercel CLI, and Claude Code skills so an agent can be productive in this codebase immediately after clone.

## Requirements

### Node.js v18+
npm is included with Node.js — no separate install needed.

- **Check your installed version:**
```bash
node -v
```
If you see `v18.x.x` or higher, you're good. If not, install or update Node.js:

- **macOS/Linux (via nvm):**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
nvm install 18
nvm use 18
```

- **Windows:** Download the installer from [nodejs.org](https://nodejs.org/)

### Supabase Project
A Supabase account, organization and project is required.

- If you don't have one already, create a free [Supabase account](https://supabase.com/dashboard/signup)
- Create a new project via the [Supabase dashboard](https://database.new).
- Note: Supabase allows 2 free projects (across any of your organizations) at a time ONLY

### Vercel Account
A Vercel account and the Vercel CLI is required.

- If you don't have one, create a [Vercel account](https://vercel.com/signup). 
- Sign in with the **same GitHub account** that will host your repo; Vercel uses auto CI/CD!
- Install the Vercel CLI globally and authenticate

```bash
npm install -g vercel
vercel login
```

## Getting Started

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

5. Configure environment variables
- Navigate to your Supabase project and click the **Connect** button in the header
- Under the **Framework** tab, select `Framework` = Next.js and `Variant` = App Router
- Under Add files, copy the contents of the `.env.local` window
- Paste the contents into `.env.example`
- Rename `.env.example` to `.env.local`

6. Configure Supabase Skills and MCP Server acces
- While in your Supabase project's **Connect** settings,
- Under the **MCP** tab, select `Client` = Claude Code and `Feature groups` = {select all}
- Click **Copy prompt**
- Start Claude Code and deliver the prompt to Claude
- Claude will: 
   - add the Supabase MCP server to the project so claude can directly control your Project (note: the Supabase MCP is nearly equivalent to a CLI tool if you configure it with all feature groups)
   - install Agent Skills for claude to become a Supabase expert

7. Configure Vercel Skills and MCP Server access
- Install the Vercel deployment skill in your terminal: `npx skills add https://github.com/vercel-labs/agent-skills --skill deploy-to-vercel`. Scope it to the project.
- Register the Vercel MCP server in your terminal: `claude mcp add --transport http vercel https://mcp.vercel.com`. On first use, Claude will prompt an OAuth flow in the browser to authorize access to your Vercel account and teams.

## Workflow

This workflow assumes that you have planned and designed your product using [DesignOS](https://buildermethods.com/design-os) and have a separate repository for those artifacts. See [**this repo first**](https://github.com/joelmsherman/Product-Planning-Design) if you haven't yet planned and designed your product. Do it now! We're spec-driven AI developers, not vibe-coders.

1. Copy the `product-plan/` folder from your planning and design repo into the root of this codebase
2. Start Claude Code
3. Choose a workflow approach: [one-shot](#one-shot) vs. [incremental](#incremental) 
4. Ship it with the [Deployment](#deployment) prompts

### One-Shot

This approach is **recommended for simpler products** with few sections and straightforward features. It's great when you need a quick prototype, or when you have time contraints and need something built fast.

1. Open `product-plan/prompts/one-shot-prompt.md`
2. Add any additional notes in the prompt, such as your tech stack preferences or any product-specific requirements/constraints
3. Deliver the prompt to Claude **in Plan mode** (shift+tab at the prompt until you see 'Plan mode')
4. Answer Claude's clarifying questions and review its final plan
5. Give Claude the go-ahead to implement - go get a coffee!
6. Review and test - `npm run dev`

### Incremental

This approach is **recommended for larger or more complicated products** when you want to build the product and review Claude's progress section by section. You start with the product foundation (design tokens, data shape/types, routing, etc.) then move on to the shell and navigation, and then implement each section one by one.

1. Open `product-plan/prompts/section-prompt.md`
2. Enter the SECTION_NAME, SECTION_ID and NN variables at the top
3. Deliver the prompt to Claude **in Plan mode** (shift+tab at the prompt until you see 'Plan mode')
4. Answer Claude's clarifying questions and review its final plan
5. Give Claude the go-ahead to implement
6. Review and test `npm run dev`
7. Move to the next section and repeat these steps until all sections are completed

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
│   ├── auth/            # login / sign-up / forgot-password routes
│   ├── protected/       # authenticated routes
│   ├── layout.tsx
│   └── page.tsx
├── components/          # React components
│   ├── ui/              # shadcn/ui primitives
│   ├── tutorial/        # onboarding walkthrough
│   └── *-form.tsx       # auth forms, theme switcher, etc.
├── lib/
│   ├── supabase/        # browser + server Supabase clients
│   └── utils.ts
├── proxy.ts             # Supabase auth cookie proxy
├── components.json      # shadcn/ui config
├── tailwind.config.ts
├── next.config.ts
├── tsconfig.json
├── .env.example
└── package.json
```