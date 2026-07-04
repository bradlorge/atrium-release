---
name: build
description: The build step archetype — implement a change in the job worktree as one step of a workflow (brief.step.skillRef = build). The implement-job discipline applied to a step-scoped change, ending in committed work + the step outcome, never a merge. Use when a workflow step asks you to make the change.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, mcp__atrium__orient, mcp__atrium__search, mcp__atrium__inspect, mcp__atrium__describe
---

# Build (workflow step)

You are running the **build** step of a workflow: implement the change in this
job's worktree. This IS the implement-job ritual — its guardrails, session-start
moves, and `.atrium/` file contract apply in full; run it. What this skill adds
is the step framing.

## The step framing

- **Scope comes from the step.** `brief.step.params.instructions` (and any
  `steer`) narrows the brief to this step's change; a `plan` step's outcome, if
  one ran, is your blueprint — follow it, and record any deviation and why in
  `.atrium/PROGRESS.md`. Do not widen scope because you spotted something
  adjacent; that is a finding for the report.
- **`success_criteria` seeds the done-when.** Fold the step's
  `success_criteria` into `.atrium/done-when.json` if the scaffold has not
  already; each item flips `met: true` only after you verified it — never on
  assumption, never to make the pipeline advance.
- **Rework is normal.** If a prior validate step failed you back here, its
  outcome names what was wrong — fix exactly that first, then re-run the checks
  that caught it, before touching anything else.

## Worked example

The step says "add the retry to the upload path per the plan"; the plan step's
outcome names the two files and proposes three done-when items. You fold them
into `.atrium/done-when.json`, make the change (noting in PROGRESS one
deviation: the helper already existed), run the checks the criteria name, flip
each item only as its check passes, and commit. The adjacent dead code you
spotted goes in the report as a finding — not into this diff.

## Finishing

- The deliverable is committed work on the job branch; the harness pushes and
  opens the PR after the session. Do NOT merge, deploy, or tag — those are
  gated by Atrium, not yours to perform.
- Finish per the **post-results** skill: what changed and how you verified it
  as the body, `result` judged against `success_criteria` (a validate step may
  still be the workflow's real judge — do not pre-claim its verdict), the PR
  branch and touched files in `refs`, `.atrium/report.md` and
  `.atrium/checks.json` in `artifacts`.
