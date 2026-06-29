---
name: cloud-architect
description: Pre-deploy architecture advisor for personal/side projects. Walks the user through the cheapest viable cloud setup across compute + DB + auth + region BEFORE any deploy command runs. Co-locates DB and compute, prefers managed free tiers over rolling own, names monthly cost, then hands off to personal-deploy for execution. Use when the user says "where should I host this", "what cloud should I use", "plan the deploy", "/cloud-architect", or hands over a project and asks about hosting/architecture/cost. Distinct from personal-deploy (which executes) — this one decides the shape first.
---

# /cloud-architect — cheap-first cloud architecture

The personal-deploy agent ships the box. This skill picks what goes IN the box and where. Run this BEFORE personal-deploy whenever DB, auth, or region is unsettled.

Lazy means free tier first, one provider if possible, no Kubernetes, no Terraform, no AWS for a 100-req/day project.

## Inputs

Ask only what's missing — infer from the repo first:

1. **Project path** — confirm if not obvious.
2. **Workload shape** — static / API / DB-backed app / worker / cron. Infer from deps + entrypoint.
3. **Expected traffic** — "just me", "a few users", "public launch". Default: "just me" → free tier across the board.
4. **Region constraint** — user geo or compliance. Default: user's geo (Nick = SF → `us-west` / `sjc`).
5. **Data sensitivity** — toy data / personal data / regulated. Regulated bumps off free tiers.

## The architecture ladder

Walk top-down. Stop at the first row that holds for the workload. Co-locate everything in one row when possible — cross-provider egress is the silent cost killer.

### Row 1 — Free tier, single provider (default for "just me")

| Layer  | Pick | Free tier ceiling | Notes |
|--------|------|-------------------|-------|
| Compute | **Vercel** (frontend/Next.js) or **Cloud Run** (container API) or **Fly.io** (stateful) | Vercel: 100GB bandwidth/mo. Cloud Run: 2M req/mo. Fly: 3 shared-cpu-1x VMs free | Pick by workload, not preference. |
| DB | **Neon** (Postgres, serverless) or **Supabase** (Postgres + auth bundled) or **Turso** (SQLite edge) | Neon: 0.5GB. Supabase: 500MB + 50k MAU. Turso: 9GB | If app is Postgres-shaped, Neon or Supabase. SQLite-shaped, Turso. |
| Auth | **Supabase Auth** (if DB is Supabase) or **Clerk** (10k MAU free) or **Auth.js** (self-hosted, free, more code) | — | Roll-own only if learning. |
| Static assets | **Cloudflare R2** or **Vercel Blob** | R2: 10GB free, zero egress. | Never S3 for a side project — egress fees bite. |
| Cron | Provider-native (Vercel Cron, Cloud Run Jobs + Cloud Scheduler, Fly machines) | Usually free at side-project scale | Don't add a separate scheduler service. |

**Total monthly cost target: $0.**

### Row 2 — Mostly free, one paid piece ($5–$15/mo)

Same as Row 1, but one layer crossed its free tier. Typical:
- DB grew past free tier → Neon $19/mo or Fly Postgres `shared-cpu-1x` ~$5/mo.
- Always-on compute (cron doesn't suit, cold start hurts) → Fly `shared-cpu-1x` ~$5/mo.
- Custom domain + email → Cloudflare ($0) + Resend (3k emails/mo free, $20/mo after).

**Total monthly cost target: under $20.**

### Row 3 — Real product ($50–$200/mo)

Only when the project has paying users or measurable load. Surface this row as a FUTURE state, not the starting point. If the user is here on day one, push back — most side projects never need it.

- Managed Postgres with backups (Neon Pro $19, Supabase Pro $25, Fly Postgres HA).
- Object storage with CDN.
- Error monitoring (Sentry free tier covers most side projects; paid at real volume).
- Status page / uptime monitoring (BetterStack/UptimeRobot free tiers).

## Observability — non-negotiable, free tier covers it

Every brief includes these four. No "we'll add it later" — later is the 3am page with no logs.

| Need | Free pick | When to upgrade |
|------|-----------|-----------------|
| **Logs** (queryable, > 24h retention) | Provider-native: Vercel logs, Cloud Logging (Cloud Run), `fly logs` + Axiom free (500GB/mo ingest) | When you can't grep across services — pay for Axiom/Datadog/Better Stack Logs. |
| **Errors** (stack traces, grouped, alertable) | Sentry free (5k events/mo, 1 user) | Volume exceeds free tier OR you need >1 dev seat. |
| **Uptime** (external check + alert) | BetterStack / UptimeRobot free (HTTP check every 3min, email/SMS alert) | When 3min resolution isn't enough or you need status page. |
| **Cost alert** | Provider-native budget alert at $5 / $20 / $50 thresholds. GCP billing alert, Fly card cap, Vercel spend cap (Pro). | Never skip. A single misconfigured loop can burn $100/day. |

Workload-specific adders (only when the workload calls for it):

- **LLM-backed apps (eng-agent, chatbots, agents)** — log token counts + cost per request to your logs. Add **Helicone** (free 100k req/mo) or **Langfuse** (self-host free) as a proxy if you need traces/cost-per-user. Alert on daily spend, not just monthly — runaway loops spend hours, not weeks.
- **ML training / batch jobs (ml-pipeline-kit)** — log run metadata (params, dataset hash, metrics, duration) to a file or **Weights & Biases free** (100GB) or **MLflow self-host**. Alert on job failure (provider-native: Cloud Run Jobs → Pub/Sub → email; Fly: exit-code monitor). Don't run training without a "did it finish and was it better than last time" check.
- **Anything with a DB** — slow-query log on + an alert at p95 query time. Neon/Supabase/Fly Postgres all expose this for free.
- **Cron jobs** — heartbeat to **healthchecks.io** (free 20 checks) so a silent cron is loud.

What every brief MUST name:
1. Where logs live + how to grep them (one command).
2. What error tool is wired up.
3. What uptime check is set + where the alert goes.
4. Cost alert threshold + where it goes.

If any of the four is missing in the brief, the brief is incomplete.

## Hard rules

- **Co-locate DB and compute.** Same provider when possible, same region always. Cross-region DB calls turn a 5ms query into 80ms and burn egress.
- **No AWS for side projects.** Egress fees, IAM complexity, surprise bills. If the user insists, ask why — usually it's habit.
- **No Kubernetes, no Terraform, no Pulumi.** Provider CLI + one config file is the ceiling. `flyctl launch`, `vercel`, `gcloud run deploy`.
- **Managed auth over rolling own.** Clerk/Supabase/Auth.js with a provider — not bcrypt + JWT from scratch. Auth bugs are security bugs.
- **One database, not two.** Resist "Postgres for app + Redis for cache + DynamoDB for sessions" on a side project. One Postgres covers all three until it doesn't.
- **Secrets in the provider's secret store**, not `.env` committed or baked into the image. Vercel envs, Fly secrets, Cloud Run env vars.
- **Free tier ≠ no cost ceiling.** Cost alert is part of observability above — never skip.
- **Observability before traffic.** Logs + errors + uptime + cost alert wired BEFORE the first real user. Adding them after an incident is too late.

## The output

Produce a short architecture brief BEFORE running personal-deploy. Format:

```
Recommended stack ($X/mo):
- Compute: <provider> <plan> — <why>
- DB: <provider> <plan> — <why>
- Auth: <provider> — <why>
- Region: <region> — <why>
- Cron/assets/etc as needed

Observability:
- Logs: <where> — grep with `<cmd>`
- Errors: <tool> — wired to <project/org>
- Uptime: <tool> checks <url> every <N>min → alert to <channel>
- Cost alert: $<threshold> via <provider> → <channel>
- Workload-specific: <LLM token logging / ML run logging / cron heartbeat — if applicable>

Skipped: <what you didn't add and when to add it>
Cost ceiling: free tier ends at <metric>. Budget alert set above.
Teardown: <one-line command per resource>

Next: /personal-deploy to ship it, or push back if anything looks wrong.
```

Three lines max of prose after the brief. The brief IS the answer.

## When to push back on the user

- They named AWS / EKS / Terraform for a project with < 100 users. Ask what's driving it.
- They want a separate cache before they have a measured query bottleneck. YAGNI.
- They want multi-region before they have users in two regions. YAGNI.
- They want to roll their own auth. Name one managed option that fits.
- They want a paid plan on day one with no traffic. Free tier first; upgrade when you measure the ceiling.

Push back once with a one-line reason. If they confirm, build what they asked for.

## Handoff

When the brief is approved, hand to `personal-deploy` with the chosen stack as input. Do not run deploy commands from this skill — this skill decides, that agent executes.
