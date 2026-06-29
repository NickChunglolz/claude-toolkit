---
name: prd
description: Drafts a Product Requirements Document (PRD) — problem, target user, success metrics, solution overview, non-goals, risks, launch plan. Notion-first for Nick (his PRDs like Plumber live there); offers to publish via Notion MCP or hand back a markdown body. Pushes back when an ERD would actually do the job. Use when the user says "draft a PRD", "write the PRD for X", "/prd", or hands over a feature/product idea that needs PM-level definition BEFORE engineering planning.
---

# /prd — Product Requirements Document

A PRD answers **what** we're building and **why**, for **whom**, and **how we'll know it worked**. It does NOT answer **how** to build it — that's an ERD. Mixing them produces a doc that's bad at both.

This skill drafts the body in markdown, then offers to publish to Notion (Nick's PRD home) or hand the markdown back.

## Push back first — does this need a PRD?

A PRD is the right tool when at least two are true:
- New product or net-new feature with user-facing impact.
- Multiple stakeholders need to align (PM + eng + design + legal/marketing/support).
- Success is measurable and the metric isn't obvious.
- The problem is fuzzy enough that scope creep is a real risk.

If the work is purely internal/engineering with no UX surface and known scope → **skip PRD, go straight to ERD** (`erd-creator`).
If the idea is still fuzzy and the problem isn't even defined → **run /kickoff first**, then come back here.
If it's a tiny feature on an existing product → **a Jira ticket + acceptance criteria is enough**.

Ask once if uncertain: *"This sounds like X — does it need a full PRD, or would a Y suffice?"* Then proceed or redirect.

## Inputs to gather (ask only what's missing)

1. **Working title** — short name. Will be prefixed `PRD: <Title>`.
2. **Problem in one sentence** — what user pain, with rough evidence (data, support tickets, user quote, your own dogfooding).
3. **Target user** — who specifically. "Everyone" is not an answer.
4. **Success metric** — one north-star + maybe one guardrail. Specific, measurable, time-bound.
5. **Constraints** — deadline, budget, team size, must-integrate-with.
6. **Existing context** — links to Glean/Notion/Linear/Jira/Figma. Pull them in parallel.
7. **Reviewers / stakeholders** — who needs to sign off, who needs to be informed.

If user gives a one-liner, propose the rest as assumptions and let them correct: *"I'm assuming target user is X, metric is Y, deadline Z — right?"*

## PRD template (sections in order, lean by default)

```markdown
# PRD: <Title>

**Status:** Draft · **Author:** <name> · **Last updated:** YYYY-MM-DD
**Reviewers:** <names> · **Related:** <ERD / Jira / Figma / Glean links>

## TL;DR
2-3 sentences. What, for whom, why now, and what success looks like.
Someone should read only this and know whether to keep reading.

## Problem
What user pain exists today, with evidence (data, quotes, dogfooding, support volume).
Why it matters — what does the user fail to do / waste time on / churn over?
Why NOW — what changed (market, tech, regulation, internal capacity)?

## Target user
Specific persona or segment. Their context, their job-to-be-done, their alternatives today.
If multiple personas, rank: who's primary, who's secondary, who's explicitly NOT served.

## Goals
Bulleted. User-facing outcomes, not features.
- Users can do X without Y.
- Time-to-task drops from N min to M min.

## Non-goals
Bulleted. What we explicitly will NOT do in this scope.
Saves a thousand future arguments. Be specific about what you're cutting and why.
- We will NOT support Z — covered by [other thing] / not enough demand / scope.

## Success metrics
- **North star:** ONE metric that goes up if this worked. Specific, measurable, time-bound.
  Example: "60% of weekly-active users complete X within 30 days of launch."
- **Guardrails:** metrics that must NOT regress. (Latency, error rate, conversion elsewhere, support volume.)
- **Counter-metrics:** what we'd see if we accidentally optimized for the wrong thing.

## Solution overview
High-level, NOT implementation. What the user sees, feels, does.
- User flow (3-7 bullets).
- Key UX moments (link Figma).
- What changes for adjacent surfaces / teams.

If multiple options were considered, name the runners-up here in 1-2 lines each + why-not.
The ERD will go deep on architecture; this is for product trade-offs only.

## Out of scope
Phases / future versions explicitly deferred. Different from non-goals (those are forever no; these are "later").

## Risks & open questions
Honest list. Not all risks have answers — listing them is the point.
- **Risk:** ... **Mitigation:** ... **Owner:** ...
- **Open question:** ... **Needs answer by:** ...

## Dependencies
Who/what needs to move for this to ship.
- Eng: <team> for X.
- Design: <name> for Figma by <date>.
- Legal/Privacy/Security: review needed if <condition>.
- Marketing/Support/Sales: launch comms, training, docs.

## Launch plan
- **Phase 0 — internal dogfood:** dates, who.
- **Phase 1 — beta:** % rollout, opt-in vs. opt-out, kill switch.
- **Phase 2 — GA:** criteria to graduate (metric thresholds, support readiness).
- **Comms:** changelog, in-app banner, email, doc updates.
- **Rollback:** what triggers it, who calls it, how fast.

## Appendix
Research links, dogfood notes, prior art, competitor scan, raw user quotes.
```

## Best practices (industry standard, applied lazy)

The lines below are the cheap-to-do, expensive-to-skip ones. Take them every time.

- **TL;DR up top, written last.** If the TL;DR is missing, the doc isn't done. Write it after the rest, refine until a stakeholder can decide from TL;DR alone.
- **Evidence beats intuition.** "Users hate X" needs a quote / ticket count / NPS verbatim / your own dogfood log. PM gut is hypothesis, not evidence — say which.
- **One north-star metric, not five.** Five metrics means no metric. Pick the one that, if it moves, means the thing worked. Guardrails sit alongside.
- **Metrics are SMART.** Specific, Measurable, Achievable, Relevant, Time-bound. "Improve engagement" fails all five. "60% of WAU complete first import within 7 days, by EOQ" passes.
- **Non-goals are mandatory.** Every PRD without non-goals scope-creeps. Name three things you're explicitly cutting.
- **Risks include the politically awkward ones.** "Legal might block this." "Adjacent team will be unhappy." Naming them in the PRD is how they get addressed, not by silence.
- **Launch ≠ ship.** Include rollback criteria + who calls it. A launch plan without a rollback plan is a hope.
- **No solution language in the Problem section.** The Problem section describes pain in user terms. If you wrote "users need a button to X", restart — that's a solution.
- **Use "we will" / "we will not", not "we should" / "we might".** PRD is a commitment doc.
- **Link every claim.** Data → dashboard. Quote → source. Figma → exact node. Stale links are findable; un-cited claims are not.
- **One author, many reviewers.** Co-authored PRDs lose voice. One person holds the pen; reviewers comment.
- **Date the doc + version it.** "Last updated YYYY-MM-DD" plus a short changelog if anything material moves after first review.
- **No engineering depth.** Data models, API contracts, infra choices belong in the ERD. If a section creeps that way, cut it and link the future ERD.

## Workflow

1. **Push-back check** — confirm a PRD is the right artifact (vs. ERD / Jira / Kickoff). If not, redirect and stop.
2. **Gather inputs** — ask only what's missing; fetch existing links in parallel.
3. **Draft body in markdown** using the template. Lean by default — empty sections marked `_TBD_` are fine; padded sections are not.
4. **Self-check before showing the user:**
   - TL;DR readable standalone?
   - North-star metric is SMART?
   - At least 3 non-goals?
   - Risks include a politically awkward one?
   - No solution language in Problem section?
   - No engineering depth that belongs in an ERD?
5. **Show draft, take feedback, iterate.** Don't publish on first draft.
6. **Publish** (on explicit OK):
   - **Notion (default for Nick)** — `mcp__notion__notion-create-pages` under the PRDs parent. Confirm parent page before creating.
   - **Markdown file** — write to repo `docs/prds/<slug>.md` if user prefers code-adjacent.
   - **Confluence** — only if user names it (Nick's PRDs live in Notion; Confluence is for ERDs).
7. **Hand off** — propose next step: `/erd-creator` for the technical design, `project-planner` for the work breakdown.

## Don'ts

- Don't write implementation details (data models, API shapes, infra) — that's the ERD.
- Don't publish without TL;DR + non-goals + SMART metric.
- Don't co-author. One author holds the pen.
- Don't pad. A 2-page PRD beats a 10-page one no one reads.
- Don't ship a PRD without a rollback plan in the launch section.

## Skip list (when NOT to add)

- Roadmap-level OKRs / "vision doc" — that's not a PRD, that's a strategy doc. Different artifact.
- "Just a button" feature — Jira ticket + acceptance criteria covers it. PRD is overhead.
- Pure infra / refactor work — go straight to ERD.
- Spike / research project — write a one-page research brief, not a PRD.
