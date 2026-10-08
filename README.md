# Forge backend

The Forge API. It serves the worker app and the employer and bank dashboards: phone and email sign-in, jobs and work sessions, wallets and withdrawals, loans, employer payments, uploads, push notifications and live updates. Payments run through Squad.

This repository is the `backend` branch of the Forge remote. The worker app is the sibling `mobile` clone and the web dashboards are the sibling `frontend` clone.

## Stack

NestJS 11, Prisma 7 with PostgreSQL through the `pg` adapter, JWT auth with argon2 hashing, class-validator for request bodies, and Swagger for the docs. The API listens on port 3000 and every route sits under `/v1`, except `/docs` and `/health`.

## Setup

You need Node 20 or newer, pnpm 9 and PostgreSQL.

```bash
git clone --branch backend --single-branch https://github.com/hackathon-by-hgs/Forge.git backend
cd backend
pnpm install
cp .env.example .env
pnpm exec prisma generate
pnpm exec prisma migrate dev
pnpm db:seed
pnpm start:dev
```

Before the migration step, create the database and role named in `DATABASE_URL`, and set `JWT_ACCESS_SECRET` and `JWT_REFRESH_SECRET` in `.env`.

- API: `http://localhost:3000/v1`
- Swagger UI: `http://localhost:3000/docs`
- Health: `http://localhost:3000/health`

Docker builds an image and runs it on port 8080 with your `.env` and an uploads volume. It does not start PostgreSQL.

```bash
docker compose up --build
```

## Route groups

| Prefix | What it covers |
| --- | --- |
| `auth` | Worker sign-in with a phone code, tokens and profile setup. |
| `dashboard/auth` | Sign-in for the employer and bank dashboards. |
| `me`, `settings`, `notifications`, `help`, `uploads` | Worker profile, settings, notifications, help and support, and file uploads. |
| `jobs`, `applications`, `sessions` | The job feed, applying, and work sessions with clock in and clock out. |
| `wallet/withdrawals`, `transactions`, `bank`, `loans` | Earnings, withdrawals, bank accounts and loans. |
| `employers`, `employer/*` | The employer API: jobs, workers, work sessions, payouts, invoices, transactions, credit, loans and analytics. |
| `search`, `ai` | Search, job recommendations and job summaries. |
| `stream` | Live updates. |
| `webhooks/squad` | Squad payment webhooks. |

## Auth and safe retries

Protected routes take `Authorization: Bearer <access token>`. Access tokens last 15 minutes and refresh tokens 30 days, set by `JWT_ACCESS_TTL` and `JWT_REFRESH_TTL`. Phone codes expire after 5 minutes, can be resent after 30 seconds and allow 5 attempts.

Money and state-changing routes accept an `Idempotency-Key` header. A repeated key replays the stored response for 24 hours, so a retry on a bad network cannot charge twice. Errors share one shape, `{ "error": { "code", "message", "details" } }`.

## Integrations

Each integration switches on when its key is present and falls back to a stub when it is not, so the API runs locally with no third-party accounts. Set `<NAME>_PROVIDER` to force a choice.

| Integration | Real when set | Without it |
| --- | --- | --- |
| Payments, payouts and SMS codes (Squad) | `SQUAD_SECRET_KEY` | Stub. |
| Email (Resend) | `EMAIL_API_KEY` | Stub. |
| File storage (Cloudflare R2) | `R2_ACCESS_KEY_ID` | Local disk in `UPLOAD_DIR`. |
| Liveness check (Smile ID) | `SMILE_PARTNER_ID` | Stub that passes every request. Do not run this in production. |
| Push notifications (Firebase Cloud Messaging) | `FCM_SERVICE_ACCOUNT_JSON` or `FCM_PRIVATE_KEY` | Stub. |
| Job summaries (Anthropic) | `ANTHROPIC_API_KEY` | Stub. |
| Job recommendations (Gemini) | `GEMINI_API_KEY` | Stub. |

## Environment

`.env.example` lists the core settings. The code reads more, so check `src/config/configuration.ts` for the full list.

| Variable | What it does |
| --- | --- |
| `NODE_ENV`, `PORT` | Environment and port. The port defaults to 3000. |
| `DATABASE_URL` | PostgreSQL connection string. |
| `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET` | Signing secrets for worker tokens. `USER_JWT_*` does the same for dashboard users. |
| `JWT_ACCESS_TTL`, `JWT_REFRESH_TTL` | Token lifetimes in seconds. |
| `OTP_TTL_SECONDS`, `OTP_RESEND_COOLDOWN_SECONDS`, `OTP_MAX_ATTEMPTS` | Phone code rules. |
| `OTP_DEBUG_EXPOSE` | When true, the code can be read from `/auth/otp/debug/:id`. Development and staging only. |
| `UPLOAD_DIR`, `UPLOAD_PUBLIC_BASE_URL`, `UPLOAD_TTL_HOURS` | Local upload settings. |
| `GEOFENCE_DEFAULT_RADIUS_M` | Clock-in radius for a job that sets none. Defaults to 200 metres. |
| `WITHDRAWAL_MIN_NAIRA`, `WITHDRAWAL_FLAT_FEE_NAIRA` | Withdrawal minimum and fee. |
| `CORS_ORIGINS`, `COOKIE_*` | Allowed browser origins and cookie settings for the dashboards. |
| `APP_BASE_URL`, `EMPLOYER_APP_BASE_URL`, `BANK_APP_BASE_URL` | Public addresses used in links and emails. |

## Scripts

| Command | What it does |
| --- | --- |
| `pnpm start:dev` | Runs the API with reload. |
| `pnpm build` | Builds into `dist`. |
| `pnpm start:prod` | Applies migrations, then runs the built API. |
| `pnpm lint` | ESLint, with fixes. |
| `pnpm test` | Unit tests. `pnpm test:e2e` runs the end-to-end test and `pnpm test:cov` adds coverage. |
| `pnpm db:push` | Pushes the schema straight to the database. Local use only. |
| `pnpm db:seed` | Loads the seed data. |
| `pnpm db:reset` | Wipes the database and seeds it again. Local use only. |
| `pnpm exec prisma studio` | Opens Prisma Studio to browse the data. |
