---
name: audit-overhead
description: Periodic cleanup pass on the Claude Code config surface — scans installed skills, agents, the auto-memory directory for the current project, and CLAUDE.md files for bloat, redundancy, drift, and stale entries. Proposes cuts without applying them. Use when the user says "audit the config", "/audit-overhead", "clean up agents/skills/memory", or wants a recurring hygiene pass (recommended monthly or every ~10 sessions).
---

# /audit-overhead — recurring config cleanup

Read-only audit of agents/skills/memory/CLAUDE.md. Proposes a cut list; applies nothing without confirmation.

## Scope

1. **Skills** — user-scope plugins under `~/.claude/skills/*/SKILL.md` (+ each plugin's `.claude-plugin/plugin.json`) and any project-scope plugins in `.claude/plugins/`
2. **Agents** — `~/.claude/agents/*.md` and `.claude/agents/*.md` in the working dir
3. **Memory** — the auto-memory directory for the current project: `~/.claude/projects/<sanitized-cwd>/memory/*.md` + `MEMORY.md`
4. **CLAUDE.md** — at working dir root (plus nested ones)
5. **Graft worktrees** (if `graft` is installed) — accumulated worktrees on disk, see section F

If you can't find the memory dir for the current project, infer the sanitized path from the working dir and ask once if uncertain.

## Flags

**A. Duplication.** Same fact (standing context, IDs, rules) in 2+ places. Source-of-truth hierarchy:
- Standing project context → `CLAUDE.md`
- Personal preferences / hard rules → `memory/`
- Agent behavior → agent file
- Workflow composition → skill file

**B. Bloat thresholds.**
- Agent > 130L → trim
- SKILL.md > 80L → split/trim
- Memory entry > 60L → likely a doc, compress
- CLAUDE.md > 200L → split into linked docs

**C. Stale.**
- `project` memories > 90 days old (verify still load-bearing)
- Memory pointing to non-existent agents/skills/files
- Drafts referenced as in-flight — check upstream; update/remove if shipped or abandoned
- Agents referencing disconnected/renamed MCP servers

**D. Drift.**
- `MEMORY.md` index entry doesn't match the file's `description:`
- CLAUDE.md claims about agents/skills don't match actual files
- CLAUDE.md hard rules conflict with `memory/feedback_*.md`

**E. Low-signal / unused.**
- Agents not invoked recently (transcripts at `~/.claude/projects/*/`)
- Skills with no clear value over a plain agent invocation
- Memory restating obvious code conventions (derivable from repo)

**F. Graft worktree sprawl.** (Skip if `command -v graft` returns nothing.)
- Run `graft ls` and check disk usage of each worktree (`du -sh` on each path).
- Flag worktrees whose branch is already merged/closed upstream (`gh pr view <branch> --json state`).
- Flag worktrees with no activity (`git -C <path> log -1 --format=%cr`) in >14 days.
- Flag total worktree count > 10 OR total disk > 20 GB — risk of memory/disk pressure.
- For each flagged worktree, recommend the appropriate `graft` cleanup command. Don't run it without confirmation.

## Run

1. List files in scope, note line counts.
2. Read each, categorize against A-E.
3. Build report (template below).
4. Ask: *"Apply these cuts? (all / select / none)"* — wait for explicit OK.
5. On confirmation, apply edits one at a time. Skip vetoed items.

## Output template

```markdown
# Config Audit — <date>

## Scope
- Skills: <N>, <L> total lines
- Agents: <N>, <L> total
- Memory: <N>, <L> total
- CLAUDE.md: <L>
- Graft worktrees: <N>, <total disk>  (or "graft not installed")

## Duplication
- **<Fact>**: appears in <files>. Canonical: <where>. Remove from <files>.

## Bloat
- `<file>` (<L>L) — <what to cut, est reduction>

## Stale
- `<file>` — <reason>. Suggest: update/remove.

## Drift
- <X says A but Y says B>. Reconcile.

## Low-signal / unused
- `<file>` — <reason>. Consider removing.

## Graft worktrees
- `<path>` (<size>, <age>, branch `<name>`, PR <state>) — Suggest: <graft cmd>

## TL;DR cut list
- [ ] <action 1>
- [ ] <action 2>

## What I didn't check
<honest boundary>
```

## Cadence

- **Monthly** default. Triggered also when: 3+ new agents/skills added, major scope change, session feels heavier than it should.
- Target: <5min, focused cut list.

## Principles

- Read-only until confirmed; show diff per edit.
- Cite file paths + line counts. No vague "feels bloated."
- Trim over delete; flag genuine zombies as removal candidates.
- Don't propose additions — this skill cuts, doesn't build. Missing pieces: one-line "noted, not in scope" at the end.

## Don'ts

- Don't apply changes without confirmation.
- Don't delete memory without user OK — even stale entries may have value.
- Don't touch the codebase — config files only.
- Don't remove a graft worktree without confirmation, even one that looks abandoned — it may hold uncommitted local work.
