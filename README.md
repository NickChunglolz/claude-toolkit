# claude-toolkit

Personal [Claude Code](https://claude.com/claude-code) plugin marketplace — reusable agents and skills that work across any project.

## What's in here

A single plugin, `nick-toolkit`, with:

### Agents

- **`project-planner`** — Plans work end-to-end from a design doc, ticket epic, or free-form goal. Outputs a sequenced, dependency-aware plan with risks and cross-team deps.
- **`project-executor`** — Drives a plan to completion. Implements, runs tests, opens draft PRs (using Conventional Commits), updates the tracker. Re-checks the plan between tasks and stops on ambiguity.
- **`idea-finder`** — Sweeps codebase / tickets / docs / Glean for improvements, gaps, and net-new ideas. Read-only; prioritized output.

### Skills

- **`/finish`** — Chains `project-planner` → confirm → `project-executor` in one flow.
- **`/audit-overhead`** — Periodic cleanup pass on your agents/skills/memory/CLAUDE.md. Also sweeps stale `graft` worktrees if `graft` is installed (skipped otherwise). Read-only; proposes cuts.

## Optional integrations

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
│   │   └── idea-finder.md
│   └── skills/
│       ├── finish/SKILL.md
│       └── audit-overhead/SKILL.md
├── CLAUDE.md.template
└── README.md
```

## License

MIT.
