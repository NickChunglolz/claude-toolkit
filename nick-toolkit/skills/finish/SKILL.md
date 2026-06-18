---
name: finish
description: Chain project-planner → project-executor in one flow. Use when the user says "finish the X project", "plan and then execute", "/finish", or wants end-to-end project completion without a manual handoff turn between planning and execution. Pass the source (design-doc URL, ticket epic key, or free-form goal) as args.
---

# /finish — plan then execute

End-to-end project completion: plan the work, then drive it. Removes the manual handoff turn between planning and execution.

## Inputs (from args or ask)

If `args` is provided, treat it as the project source. Otherwise ask once:

- **Source** — design-doc URL/page ID, ticket epic key (e.g. `PROJ-1234`), or free-form goal
- **Session scope** — "next task only" / "next N" / "run until blocked" / "finish whole project" (default: until blocked)
- **PR mode** — "draft PR per task" (default) / "single PR" / "no PR yet"

Don't ask anything that's clear from `args`.

## Flow

1. **Plan** — Invoke the `project-planner` agent via the Agent tool with the source. Wait for the plan.
2. **Confirm** — Show the plan to the user. Ask: *"Plan looks right? Proceed to execute (yes / edit / abort)?"*
   - If `edit`: relay the user's edits back to the planner agent for revision, loop step 2.
   - If `abort`: stop.
   - If `yes`: continue.
3. **Execute** — Invoke the `project-executor` agent, passing the approved plan + session scope + PR mode.
4. **Report** — Surface the executor's session summary back to the user verbatim.

## Notes

- If the planner returns major risks/blockers (e.g., "design doc has unresolved Open Questions", "code reality contradicts plan"), STOP at step 2 even if the user pre-approved — call out the risk before proceeding.
- If the source is a design doc that looks incomplete, suggest running `erd-reviewer` (if installed) first instead of jumping to plan.
- Plan and executor agents both inherit `CLAUDE.md` standing context — don't re-explain it.
- This skill is the composition layer. The two agents do the actual work; this just removes the handoff turn.
