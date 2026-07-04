---
name: finalise
description: The finalise step archetype — tidy the work, stage the PR, and write the hand-off summary as a workflow's closing step (brief.step.skillRef = finalise). Leaves the branch presentation-ready and the operator fully oriented; never merges. Use when a workflow step asks you to wrap the work up for human hands.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, mcp__atrium__inspect, mcp__atrium__describe
---

# Finalise (workflow step)

You are running the **finalise** step of a workflow: the work is done — make it
receivable. Tidy the branch, stage the PR material, and write the hand-off so
the operator (and Atrium) can take it from here without archaeology. The
implement-job ritual's contract applies; this skill is only the shape of the
step.

## The discipline

- **Tidy, don't extend.** Remove debug leftovers, stray files, and commented-out
  scaffolding; make the final commit state clean. NO new functionality, no
  "while I'm here" improvements — if the work is incomplete, that is a prior
  step's failure to report honestly, not this step's gap to quietly fill.
- **Never merge.** The deliverable is an OPEN PR — the harness pushes the branch
  and opens it; merge/deploy are gated by Atrium. Do not merge, do not tag, do
  not deploy.
- **Write for the receiver.** The hand-off is read by someone with none of your
  context: what changed and why, how it was verified (point at the validate
  outcome, don't re-litigate it), what is deliberately out of scope, what to
  watch after merge.

## The moves

1. Read the prior steps' outcomes — the build's report and the validate
   verdict are the facts your hand-off summarizes.
2. Sweep the worktree: `git status` clean, no debug artifacts, commits coherent
   (squash-tidy only if history is noise; never rewrite what tells a story).
3. Stage the PR material in `.atrium/report.md`: title, description, the
   verification evidence, reviewer notes and out-of-scope list.
4. Confirm every `done_when` item's state is truthful — finalise inherits the
   honesty of the steps before it and must not launder a `partial` into a done.

## Worked example

Build and validate both succeeded. You sweep the worktree (`git status` shows
a stray `debug.log` — removed), squash two "wip" commits into their parent,
write `.atrium/report.md` with the PR title/description, the validate step's
per-criterion table as the verification evidence, and an out-of-scope list
("locale handling untouched — see finding in explore"). Every `done_when`
state is re-checked truthful; nothing merged, nothing new built.

## Finishing

Finish per the **post-results** skill: the hand-off summary as the body, the
branch name and validate-outcome refs in `refs`, `.atrium/report.md` in
`artifacts`. `result: success` means "ready for human review" — nothing more.
