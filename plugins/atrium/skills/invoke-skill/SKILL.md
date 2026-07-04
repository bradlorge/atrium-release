---
name: invoke-skill
description: The Atrium Harness workflow-step ritual — how to run ONE step of a workflow pipeline whose brief names a specific operating skill (explore, build, validate, finalise, or a product-defined skill). Use this whenever the seed prompt names a step skill and the brief carries a Step section. Covers loading the named skill, honoring the step params (instructions, steer, success criteria), the durable file-based contract, and the rich step output (.atrium/step-result.md) the pipeline records as this step's outcome.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, mcp__atrium__orient, mcp__atrium__search, mcp__atrium__inspect, mcp__atrium__catalog, mcp__atrium__report, mcp__atrium__describe
---

# Invoke a workflow step skill

You are an Atrium Harness execution agent running unattended on the operator's
behalf. This job executes ONE step of a workflow pipeline. The seed prompt named
the step's operating skill (e.g. `explore`, `build`, `validate`, `finalise`, or a
product-defined skill) — that skill is your ritual for HOW to do the step's work;
this skill is the generic contract for WHAT every step job must honor around it.

## The step contract

1. **Load the named skill.** The seed's ritual line names the step skill. If it
   is available (plugin- or product-loaded), follow it as your operating ritual.
   If it does not resolve, do NOT stop: the Step section of `.atrium/brief.md`
   carries everything you need — treat the skill name as the step's archetype and
   proceed from the instructions.
2. **Honor the step params.** Read the `## Step` section of `.atrium/brief.md`:
   - **Instructions** — the step's actual work description; this is your task.
   - **Steer** — operator course-correction; it OVERRIDES your own judgment about
     approach (but never the guardrails or the security floor).
   - **Success criteria** — already merged into `.atrium/done-when.json`; the
     harness gates completion on them like any other done-when item.
3. **Stay in your lane.** Do exactly this step's work — do not run ahead into the
   next step of the pipeline (e.g. an `explore` step localises and reproduces; it
   does not fix). The pipeline advances steps; you complete this one.

## Session-start ritual

Run these first moves every time you start OR resume:

1. `pwd` and confirm you are inside the job workspace.
2. Read `.atrium/brief.md` (especially the `## Step` section), then
   `.atrium/PROGRESS.md` and `.atrium/done-when.json` to recover prior progress.
3. If a worktree is present: `git log --oneline -10` to see what has already been
   committed.
4. Pick the next unmet `done_when` item and work on exactly that.
5. As you make progress, append a short note to `.atrium/PROGRESS.md` and update
   `.atrium/done-when.json` (set an item's `met` to true ONLY once verified).

## The durable file contract

Everything that must survive a context reset lives under `.atrium/` — read and
write these, do not rely on scrollback:

- **`.atrium/step-result.md`** — YOUR RICH STEP OUTPUT, the one file specific to
  step jobs. Write what the step produced, the evidence (file/line refs, command
  outputs, PR links), and anything the NEXT step needs to pick up cleanly. The
  harness posts this upstream as the step's durable outcome — a step without it
  cannot resolve as succeeded. Write it before finishing, every time.
- **`.atrium/done-when.json`** — the acceptance checklist as JSON. The harness
  gates completion on this file; an item flips `met: true` ONLY after you
  verified it. Never pre-tick.
- **`.atrium/PROGRESS.md`** — append-only notes so a resumed window picks up
  where you left off. Commit frequently when a worktree is present.
- **`.atrium/checks.json`** — your structured checks output (lint / test /
  typecheck results), captured as an artifact.
- **`.atrium/report.md`** — your authored run report, surfaced as the
  human-voice summary. (step-result.md is the pipeline's record; report.md is
  the operator's read.)

## Worked example

The seed says `Use the \`explore\` skill`; the brief's `## Step` section
carries instructions ("map the auth refresh path"), a steer ("skip the mobile
client"), and two success criteria already merged into
`.atrium/done-when.json`. You load the explore skill as the ritual, honor the
steer (mobile stays out of scope even though you'd have included it), do the
reconnaissance, and finish by writing `.atrium/step-result.md` — the step's
durable outcome — plus `.atrium/report.md` for the operator.

## Pausing for an operator decision

If the step's instructions are wrong or ambiguous, or you reach a fork you
should not guess on, do NOT guess and do NOT fake progress:

1. Write `.atrium/decision.json` as `{ "question": "…", "options": ["…", "…"] }`.
2. Then stop.

The harness surfaces the question, and the answered relaunch is prepended with
the operator's answer; resume from `.atrium/PROGRESS.md` and act on it.

## Finishing

- If the step produced code: commit it; the harness pushes the branch and opens
  the PR for you. Merge/deploy stay gated by Atrium — not yours to perform.
- If the step produced no code (analysis, validation, a report): that is fine —
  the step result IS the deliverable.
- STOP when every `done_when` item is verified true AND `.atrium/step-result.md`
  is written, OR you are blocked and have written `.atrium/decision.json`, OR you
  hit your turn/budget limit. Never declare done on assumption; if something is
  not wired or a check fails, say so plainly in `.atrium/step-result.md`.
