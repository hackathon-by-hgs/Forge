# Forge web

The Forge web apps: a public landing site and two dashboards. Employers (wholesalers, factories, retailers) post jobs, manage workers and work sessions, handle payments and credit, and read analytics. Banks (credit officers and risk analysts) review borrowers and loan applications and watch the loan portfolio and its performance. All three apps share one UI library and one types package, and all talk to the same Forge API.

This repository is the `frontend` branch of the Forge remote. The API is the sibling `backend` clone and the worker app is the sibling `mobile` clone.

| App | Who uses it | Dev port |
| --- | --- | --- |
| `apps/landing` | Everyone. Marketing site with links to both dashboards. | 3002 |
| `apps/employer-web` | Employers. | 7001 |
| `apps/bank-web` | Credit officers and risk analysts. | 7001 |

`employer-web` and `bank-web` both default to port 7001 in their `dev` scripts, so `pnpm dev` at the root cannot run both at once. Change the port in one app's `package.json`, or run one app at a time with `pnpm --filter employer-web dev`.

## Stack

Next.js 14 with the App Router, TypeScript, Tailwind CSS, and a Turborepo and pnpm workspace. Shared code lives in `packages/`.

## Setup

You need Node 20 or newer and pnpm 9 or newer.

```bash
git clone --branch frontend --single-branch https://github.com/hackathon-by-hgs/Forge.git frontend
cd frontend
pnpm install
cp apps/landing/.env.example apps/landing/.env
cp apps/employer-web/.env.example apps/employer-web/.env
cp apps/bank-web/.env.example apps/bank-web/.env
pnpm dev
```

## Environment

| Variable | App | What it does |
| --- | --- | --- |
| `NEXT_PUBLIC_API_BASE_URL` | `employer-web`, `bank-web` | The Forge API address. The example points at the deployed API, `https://forgebe-production.up.railway.app`. Use `http://localhost:3000` for the local `backend` clone. |
| `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` | `employer-web` | Optional. Without it, job maps show a placeholder. Restrict the key by referrer in Google Cloud. |
| `NEXT_PUBLIC_EMPLOYER_URL` | `landing` | Where the employer button on the landing site links. |
| `NEXT_PUBLIC_BANK_URL` | `landing` | Where the bank button on the landing site links. |

## Scripts

Run these from the repository root. Turborepo runs each one in every workspace and caches the results.

| Command | What it does |
| --- | --- |
| `pnpm dev` | Starts the apps in development. |
| `pnpm build` | Builds every app. |
| `pnpm lint` | Lints every app. |
| `pnpm typecheck` | Type checks every app. |
| `pnpm test` | Runs vitest. There are no test files yet, so it passes with none. |
| `pnpm format` | Formats the code with Prettier. `pnpm format:check` only checks. |
| `pnpm clean` | Removes build output and `node_modules`. |

## Layout

| Path | What it is |
| --- | --- |
| `packages/ui` | Shared components and the Tailwind preset. |
| `packages/types` | Shared domain types and schemas. |
| `packages/mock-data` | Mock data generators with loading, empty and error states. |
| `packages/tsconfig`, `packages/eslint-config`, `packages/prettier-config` | Shared tooling config. |

## Deployment

- Railway builds from the repository root. `nixpacks.toml` makes Nixpacks use Node 20.
- `docker-compose.yml` builds `bank-web` and `employer-web` from `apps/<app>/Dockerfile`.
- `ecosystem.config.mjs` is a PM2 template for a VPS. Replace the app name and the `cwd` path before using it.
- `.github/workflows/bank-web.yml` and `employer-web.yml` lint, type check, test, build and build a Docker image. They run on pushes and pull requests to `master` or `main`, so change the branch filter to `frontend` if you want them to run on this branch.
