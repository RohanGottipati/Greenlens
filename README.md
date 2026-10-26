# GreenLens AI

Measure the environmental and financial footprint of your organization's AI usage — carbon, water, license utilization and spend — and turn it into board-ready decisions and ESG disclosures.

## What it does

GreenLens connects to an organization's AI providers using aggregate admin APIs (no individual prompts or messages are collected), runs a multi-step analysis pipeline and presents the results in a dashboard:

- **Integrations** — Microsoft 365 (Microsoft Graph reports, OAuth), Google Workspace (licensing, directory and audit reports, OAuth) and OpenAI (organization admin key, Usage API).
- **Analysis pipeline** — usage analyst → statistical analysis → carbon & water accountant → license intelligence → strategic translator → synthesis. Jobs are persisted in Supabase and advanced step by step while the client polls `/api/pipeline/status`.
- **Dashboard** — Overview, Model Efficiency, Carbon & Water, Licenses, Incentives, Benchmark, Decisions and ESG Export pages, with partial reports when a data source is unavailable.
- **ESG export** — PDF report generated with `@react-pdf/renderer` (`/api/reports/export`).
- **Auth** — Supabase Auth (email magic link plus Microsoft/Azure, Google and GitHub OAuth providers); `/dashboard` and `/onboarding` are protected by `src/proxy.ts`.

## Tech stack

- Next.js 16 (App Router) + React 19 + TypeScript
- Tailwind CSS 4, Recharts, lucide-react
- Supabase (Postgres + Auth) via `@supabase/ssr` / `@supabase/supabase-js`
- Backboard API (primary LLM for the strategic translator) and Google Gemini (`@google/generative-ai`, fallback + ESG narrative)
- Tests: Node's built-in test runner via `tsx`

## Project structure

```
src/
  app/
    (auth)/            login & signup pages
    (dashboard)/       dashboard pages and layout
    onboarding/        company details, integrations, confirm
    api/               integrations OAuth/key connect, pipeline trigger/status, reports, export, account
    testing/           Backboard debug page (its /api/testing/backboard route needs dev or ENABLE_BACKBOARD_TEST_ROUTE)
  components/          dashboard, landing and onboarding UI
  lib/
    agents/            pipeline agents + orchestrator
    analysis/          job state, polling, stat analysis, model classification
    integrations/      Microsoft, Google, OpenAI clients and token handling
    calculations/      carbon and water models
    reports/ billing/  report shaping, availability, modeled spend
    backboard/ gemini/ supabase/  service clients
    demo/              fake provider data returned for DEMO_* sentinel tokens
  analysis/pipeline.py reference Python version of the statistical analysis (not used at runtime)
  proxy.ts             auth guard for /dashboard and /onboarding
tests/                 unit tests (*.test.ts)
DATABASE_FULL_SETUP.sql  canonical Supabase schema, seeds and RLS
supabase-migrations/     incremental repairs for older databases
reset-*.sql              helpers to wipe tenant data
Implementation_Plan.md   original hackathon implementation plan and product framing
```

## Prerequisites

- Node.js 20.9+ and npm (required by Next.js 16)
- A Supabase project
- API credentials for the providers you want to use (Backboard, Gemini, Microsoft Entra app, Google Cloud OAuth client)

## Setup

```bash
git clone https://github.com/RohanGottipati/Greenlens.git
cd Greenlens
npm install
cp .env.example .env.local   # then fill in the values
```

Database: run `DATABASE_FULL_SETUP.sql` in the Supabase SQL editor (safe to re-run). For an existing older database, the scripts in `supabase-migrations/` add missing columns. `supabase-setup.sql` is a legacy version kept for reference.

OAuth redirect URIs to register with the providers:

- Microsoft: `<NEXT_PUBLIC_SITE_URL>/api/integrations/microsoft/callback`
- Google: `<NEXT_PUBLIC_SITE_URL>/api/integrations/google/callback`
- Supabase Auth login redirect: `<NEXT_PUBLIC_SITE_URL>/auth/callback`

Run the dev server:

```bash
npm run dev
```

Open http://localhost:3000.

## Environment variables

See `.env.example`.

| Variable | Purpose |
| --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase anon key (browser/server clients) |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service role key (server-side admin client) |
| `BACKBOARD_API_KEY` | Backboard API key for the strategic translator agent |
| `GEMINI_API_KEY` | Gemini API key (translator fallback, ESG export narrative) |
| `MICROSOFT_CLIENT_ID` / `MICROSOFT_CLIENT_SECRET` / `MICROSOFT_TENANT_ID` | Microsoft Graph OAuth app (tenant defaults to `common`) |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | Google Workspace OAuth client |
| `NEXT_PUBLIC_SITE_URL` | Public base URL used to build OAuth redirect URIs |
| `ENABLE_GOOGLE_USAGE_SPIKE` | `true` enables exploratory Google Gemini activity collection during analysis (off unless set, even in development) and exposes `/api/reports/[id]/google-usage-spike` outside development |
| `ENABLE_BACKBOARD_TEST_ROUTE` | `true` enables `/api/testing/backboard` outside development |

GitHub/Google/Microsoft *login* providers are configured in the Supabase dashboard, not via app env vars.

## Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the Next.js dev server |
| `npm run build` | Production build |
| `npm start` | Serve the production build |
| `npm run lint` | ESLint |
| `npm test` | Unit tests (`tsx --test ./tests/*.test.ts`) |

The optional Python reference script uses `requirements.txt` (`pip install -r requirements.txt`); the app does not need Python.
