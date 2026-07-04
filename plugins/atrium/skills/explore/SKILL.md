---
name: explore
description: The explore step archetype — codebase/problem reconnaissance for a workflow step (brief.step.skillRef = explore). Read-only investigation of the repo and the problem space that ends in a written report, not code changes. Use when a workflow step asks you to understand something before anyone builds it.
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, mcp__atrium__orient, mcp__atrium__search, mcp__atrium__inspect, mcp__atrium__report, mcp__atrium__describe
---

# Explore (workflow step)

You are running the **explore** step of a workflow: reconnaissance, not
construction. The step's focus arrives in `brief.step.params`
(`instructions`, `steer`, `success_criteria`); the base session ritual and the
durable `.atrium/` file contract are the implement-job skill's — follow them.
This skill is only the shape of the step.

## The discipline

- **Read, don't write.** No source changes, no refactors-while-you're-in-there.
  Your only deliverables are `.atrium/` files. If you find something that must
  change, that is a *finding* for the report, not an edit.
- **Trace, don't guess.** Every claim in the report cites its evidence: a file
  path + line, a command you ran and its output, a doc you read. "Probably"
  belongs in an explicit open-questions list, not in a finding.
- **Bound the sweep.** `params.instructions` names the territory; stay on it.
  Note adjacent hazards you trip over in one line each — do not chase them.

## The moves

1. Read `brief.step.params.instructions` and `success_criteria` — they define
   what "understood" means for this step.
2. Map the territory: entry points, the data flow, the seams, the tests that
   already cover it. Run the code where running it answers a question.
3. Record findings in `.atrium/PROGRESS.md` as you go — file:line refs,
   command outputs, dead ends (a ruled-out path is a finding too).
4. Distill into a report the next step can act on without re-deriving your
   work: what exists, how it behaves, where the risk is, what you could not
   determine and why.

## Worked example

The step asks "how does session refresh work and where could reuse detection
hook in?" You trace the token path (naming file:line for mint, refresh,
revoke), run the auth tests to confirm the rotation behavior, and rule out the
middleware as a hook point (with the output that proves it). The report gives
the flow, the one viable seam, and an open question ("could not determine how
mobile clients refresh — no fixture") — and changes no code.

## Finishing

Finish per the **post-results** skill: `.atrium/step-result.md` with the report
as the body, `result` judged against the step's `success_criteria`, and every
finding's evidence in `refs`. An honest "could not determine X" beats a
confident guess — the next step builds on this.
