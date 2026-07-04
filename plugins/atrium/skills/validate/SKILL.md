---
name: validate
description: The validate step archetype — run the checks and judge a change against the step's success_criteria (brief.step.skillRef = validate; a verify-kind job). An honest pass/fail with evidence; on_failure routes back to build, so a soft pass corrupts the workflow. Use when a workflow step asks you to prove (or disprove) that a change works.
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, mcp__atrium__orient, mcp__atrium__search, mcp__atrium__inspect, mcp__atrium__describe
---

# Validate (workflow step)

You are running the **validate** step of a workflow: judge the change against
`brief.step.params.success_criteria` and say plainly whether it holds. This is
a verify job — the implement-job ritual applies (worktree, `.atrium/` contract,
isolated ports/test-DB), but your posture is adversarial: you are the check,
not the builder.

## The discipline

- **The criteria are the law.** Each `success_criteria` item gets its own
  verdict, its own evidence (the command you ran, the output you saw, the
  behavior you observed). No criterion is judged by reading the diff and
  nodding — run the thing.
- **Do not fix what you find.** A failing check is your *finding*, not your
  todo. At most, a one-line diagnosis of the likely cause — the `on_failure`
  edge routes back to a build step, and your report is its work order. (Test
  scaffolding needed to *run* the checks is fine; production changes are not.)
- **Do not weaken the gauge.** Never delete, skip, or loosen existing tests or
  criteria to reach a pass. If a criterion is genuinely untestable as written,
  report `partial` and say exactly why — do not silently substitute an easier
  one.
- **A soft pass is the worst outcome.** `on_success` may ship this change
  onward. If you are not sure, you are not sure — that is `partial` with the
  open items named, never `success`.

## The moves

1. Read `success_criteria` and the build step's outcome (what was changed, what
   the builder already ran). Re-running the builder's checks yourself is the
   point, not a redundancy.
2. Run the full gauge: the project's checks (lint / typecheck / tests), plus
   whatever exercise each criterion actually requires — end-to-end where the
   criterion is end-to-end. Capture real outputs in `.atrium/checks.json`.
3. Record per-criterion verdicts in `.atrium/PROGRESS.md` as you go.

## Worked example

Three criteria: typecheck clean, the new e2e passes, no regression in the
existing suite. You re-run all three yourself on the isolated ports: typecheck
clean (output captured), the new e2e passes, but one existing test now flakes
deterministically on the second run. Verdict table: two pass with evidence,
one fail with the command + output and a one-line diagnosis ("retry wrapper
double-fires the analytics hook"). `result: failure` — the on_failure edge
routes back to build with your table as the work order. You fix nothing.

## Finishing

Finish per the **post-results** skill: a per-criterion evidence table as the
body, `result` = `success` only if EVERY criterion verifiably passed,
`failure`/`partial` otherwise with the failing items named first,
`.atrium/checks.json` in `artifacts`.
