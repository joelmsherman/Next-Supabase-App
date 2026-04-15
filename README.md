# Next.js / Supabase Starter App Template

## Introduction

A Next.js + Supabase starter template optimized for **agent-first development**. It bundles the [Supabase Next.js quickstart](https://supabase.com/docs/guides/getting-started/quickstarts/nextjs) with the Supabase MCP server and Claude Code skills so an agent can be productive in this codebase immediately after clone.

Under the hood it ships the same stack as the [official Supabase demo](https://demo-nextjs-with-supabase.vercel.app/):

- Works across the entire [Next.js](https://nextjs.org) stack — App Router, Pages Router, Proxy, Client, Server
- `supabase-ssr` for cookie-based Supabase Auth
- Password-based authentication block from the [Supabase UI Library](https://supabase.com/ui/docs/nextjs/password-based-auth)
- [Tailwind CSS](https://tailwindcss.com) styling
- [shadcn/ui](https://ui.shadcn.com/) components
- Optional one-click deploy via the [Supabase Vercel Integration](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fvercel%2Fnext.js%2Ftree%2Fcanary%2Fexamples%2Fwith-supabase&project-name=nextjs-with-supabase&repository-name=nextjs-with-supabase) (env vars assigned automatically)

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
If you don't have one already, create an account and organization in Supabase, and initialize a new project via the [Supabase dashboard](https://database.new).

## Getting Started

1. Clone this template to a local repository `repo-name` of your choice and install dependencies:

```bash
# Clone the template
git clone https://github.com/joelmsherman/Next-Supabase-App.git /path/to/your/<repo-name>
cd /path/to/your/<repo-name>

# Install dependencies
npm install
```

2. Configure environment variables
- Navigate to your Supabase project and click the **Connect** button in the header
- Under the **Framework** tab, select `Framework` = Next.js and `Variant` = App Router
- Under Add files, copy the contents of the `.env.local` window
- Paste the contents into `.env.example`
- Rename `.env.example` to `.env.local`

3. Empower Claude with Supabase Skills and MCP Server access
- While in your Supabase project's **Connect** settings,
- Under the **MCP** tab, select `Client` = Claude Code and `Feature groups` = {select all}
- Click **Copy prompt**
- Start Claude Code and paste the prompt to Claude
- Claude will 
   - add the Supabase MCP server to the project so claude can control your database
   - install Agent Skills for claude to become a Supabase expert

## Workflow

This workflow assumes that you have planned and designed your product using [DesignOS](https://buildermethods.com/design-os) and have a separate repository for those artifacts. See [**this repo first**](https://github.com/joelmsherman/Product-Planning-Design) if you haven't yet planned and designed your product. Do it now!

1. Copy the `product-plan/` folder from your planning and design repo into the root of this codebase
2. Start Claude Code
3. Choose a workflow approach: [one-shot](#one-shot) vs. [incremental](#incremental) 

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

## Deployment (Vercel)

Vercel's Git integration gives you automatic CI/CD out of the box: every push to `main` ships production, every PR gets its own preview URL.

1. **Push your cloned repo to GitHub** (if you haven't already).
2. **Import the repo into Vercel**
   - Go to [vercel.com/new](https://vercel.com/new) and select your GitHub repo
   - Framework preset: Next.js (auto-detected)
   - Leave build/output settings at defaults
   - Do **not** deploy yet — environment variables aren't wired up
3. **Link Supabase via the Vercel Integration**
   - In your Supabase project's **Connect** settings, open the **Vercel** tab (or visit the [Supabase Vercel Integration](https://vercel.com/integrations/supabase))
   - Install the integration, select your Vercel project, and link it to this Supabase project
   - `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` are injected into Production, Preview, and Development environments automatically
4. **Deploy**
   - Back in Vercel, click **Deploy**. Production builds from `main`.
   - Any subsequent push to `main` → production deploy
   - Any PR against `main` → preview deploy, URL posted on the PR
5. **Configure the Supabase redirect URL allow-list**
   - In Supabase → **Authentication → URL Configuration**, add your Vercel production domain and the `*.vercel.app` preview wildcard so auth redirects work on preview deploys

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