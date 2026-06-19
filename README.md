# claude-toolkit

Personal [Claude Code](https://claude.com/claude-code) plugin marketplace — reusable agents and skills that work across any project.

## What's in here

A single plugin, `nick-toolkit`, with:

### Agents

- **`project-planner`** — Plans work end-to-end from a design doc, ticket epic, or free-form goal. Outputs a sequenced, dependency-aware plan with risks and cross-team deps.
- **`project-executor`** — Drives a plan to completion. Implements, runs tests, opens draft PRs (using Conventional Commits), updates the tracker. Re-checks the plan between tasks and stops on ambiguity.
- **`idea-finder`** — Sweeps codebase / tickets / docs / Glean for improvements, gaps, and net-new ideas. Read-only; prioritized output.
- **`personal-deploy`** — Deploys a side project to Cloud Run, Vercel, or Fly.io. Picks the target by workload shape and cost (no AWS / Terraform / k8s for a 100-req/day project), generates minimal config if missing, shows the plan with cost estimate, runs the deploy after confirmation, smoke-tests, and returns URL + teardown command.

### Skills

- **`/kickoff`** — Consultant-style discovery for a fuzzy idea / project / ERD / PRD. Reflects what you said, asks 2-3 highest-value follow-ups (not a form), then synthesizes a structured Kickoff Brief and suggests a next step (brainstorm / project-planner / erd-creator / PRD / hold). Use when an idea isn't yet shaped enough to plan.
- **`/finish`** — Chains `project-planner` → confirm → `project-executor` in one flow.
- **`/audit-overhead`** — Periodic cleanup pass on your agents/skills/memory/CLAUDE.md. Also sweeps stale `graft` worktrees if `graft` is installed (skipped otherwise). Read-only; proposes cuts.
- **`/verify-regression`** — Proves a bug-fix test is a real regression test by running it against the old code (must FAIL) and the new code (must PASS). Catches tests that assert the wrong invariant, mock the broken path, or depend on infrastructure that didn't exist before the fix.
- **`/scan-secrets`** — Read-only audit for leaked secrets in a file, directory, diff, or staged change. Wraps `gitleaks` if installed, otherwise regex sweep (AWS, GCP, GitHub, Slack, OpenAI, JWT, PEM). Always redacts values in output. Pairs with the secret-hygiene rules baked into every agent in this toolkit.

## Optional integrations

- **[`ponytail`](https://github.com/DietrichGebert/ponytail)** — recommended companion. Always-on skill that makes the agent walk a "lazy ladder" before writing code (YAGNI → stdlib → native → installed dep → one-liner → minimum). Benchmarked at -54% LOC / -20% cost / -27% time on real Claude Code tasks, with safety guards kept intact. `project-executor` references the same ladder in its prompt, so the principle holds even without ponytail loaded — but the hook-level enforcement is worth installing:
  ```
  /plugin marketplace add DietrichGebert/ponytail
  /plugin install ponytail@ponytail
  ```
- **[`context7`](https://github.com/upstash/context7)** — recommended companion. Pulls live, version-specific library docs into the session instead of relying on the model's training cutoff. Auto-invokes on library/API mentions; manual lookup via `/context7:docs <library> <query>`. Pairs well with `project-executor` when touching fast-moving libraries (Next.js, Prisma, gRPC bindings, etc).
  ```
  /plugin marketplace add upstash/context7
  /plugin install context7@context7-marketplace
  ```
- **[`code-review`](https://claude.com/plugins/code-review)** (Anthropic official) — recommended companion. Runs 5 parallel Sonnet agents on your diff (CLAUDE.md compliance, bug detection, git-blame history, PR history, comments) with confidence scoring 0–100; only surfaces findings above threshold. Run before flipping `project-executor`'s draft PR to ready.
  ```
  /plugin install code-review@claude-plugins-official
  ```
- **`graft`** — git-worktree helper. If installed, `project-executor` creates an isolated worktree per task (cheap parallel work, easy rollback) and `/audit-overhead` flags worktrees that are merged/closed/stale or pushing disk usage past a threshold. Without `graft`, both agents fall back gracefully — nothing breaks.
- **Atlassian MCP** — enables Jira ticket transitions and Confluence doc fetches in `project-planner` / `project-executor` / `idea-finder`. Without it, they skip those steps and operate from chat context only.
- **Glean MCP** — enables semantic search across Slack/docs/PRs in `idea-finder`. Without it, the sweep narrows to local code + tickets/docs.

## Install

```bash
claude plugin marketplace add NickChunglolz/claude-toolkit
claude plugin install nick-toolkit@nick-marketplace
```

Then reload plugins (`/reload-plugins` inside Claude Code) or restart the CLI.

## Set up the standing context

Each agent reads a `CLAUDE.md` at your working-dir root for project-specific facts (repos, ticket project key, branch conventions, hard rules, etc.) so you don't re-explain them every session.

Copy [`CLAUDE.md.template`](./CLAUDE.md.template) to your project root, rename it to `CLAUDE.md`, and fill in the slots. Delete sections that don't apply (e.g. drop the Jira block if your team doesn't use Jira).

The agents degrade gracefully if a section is missing — e.g. `project-executor` will skip Jira transitions and just report in chat if no ticket system is configured.

## Conventional commits

`project-executor` writes commits and PR titles in [Conventional Commits](https://www.conventionalcommits.org/) style:

```
<type>: <short imperative description> (<TICKET-KEY>)
```

Types in rotation: `feat`, `fix`, `hotfix`, `chore`, `refactor`, `perf`, `test`, `docs`. The ticket key suffix is appended only if your `CLAUDE.md` configures a tracker.

## Updating the plugin

```bash
claude plugin marketplace update nick-marketplace
claude plugin update nick-toolkit@nick-marketplace
```

## Layout

```
.
├── .claude-plugin/
│   └── marketplace.json
├── nick-toolkit/
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── agents/
│   │   ├── project-planner.md
│   │   ├── project-executor.md
│   │   ├── idea-finder.md
│   │   └── personal-deploy.md
│   └── skills/
│       ├── finish/SKILL.md
│       ├── kickoff/SKILL.md
│       ├── audit-overhead/SKILL.md
│       ├── verify-regression/SKILL.md
│       └── scan-secrets/SKILL.md
├── CLAUDE.md.template
└── README.md
```

## License

MIT.
