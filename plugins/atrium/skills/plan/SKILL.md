---
name: plan
description: The plan step archetype — design an approach for a workflow step (brief.step.skillRef = plan) and propose the done-when that will judge the build. Turns prior steps' findings into a concrete, verifiable plan without writing production code. Use when a workflow step asks how something should be done before it is done.
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, mcp__atrium__orient, mcp__atrium__search, mcp__atrium__inspect, mcp__atrium__report, mcp__atrium__describe
---

# Plan (workflow step)

You are running the **plan** step of a workflow: turn what is known (the brief,
prior steps' outcomes, the codebase) into an approach a build step can execute
and a validate step can judge. No production code — the deliverable is the
plan. The base ritual and `.atrium/` contract are the implement-job skill's;
this skill is only the shape of the step.

## The discipline

- **Plan against reality.** Read the actual code paths the plan touches; a plan
  written from the brief alone is a guess with formatting. Cite file:line for
  every "change X" item.
- **Propose the done-when.** The plan's most important output is the acceptance
  checklist the build will be judged by: each item concrete, verifiable by a
  command or an observation, and honest about what it does NOT prove. A
  done-when a machine can check beats one a human must interpret.
- **Name the forks.** Where two approaches are defensible, pick one, say why,
  and record the road not taken. If the fork is genuinely not yours to call —
  it changes scope, cost, or user-visible behavior beyond the brief — pause for
  the operator (`.atrium/decision.json`, per the implement-job ritual) instead
  of guessing.

## The moves

1. Read `brief.step.params.instructions` and any prior step outcomes referenced
   in the brief — explore/gather-feedback reports are your ground truth.
2. Read the code the plan will touch. Note constraints (frozen contracts,
   invariants, test coverage) that shape the approach.
3. Write the plan: the approach in one paragraph, the change list (each item
   with its file anchors), the risks and their mitigations, and the **proposed
   done-when** as a checklist the build step can adopt verbatim.
4. Sanity-check it: could a competent agent with no scrollback execute this
   plan from the document alone? If not, it is not done.

## Worked example

The explore step mapped the upload path; the brief asks to plan retry
handling. You read the two modules it named, pick exponential backoff over a
queue (recording why, and the road not taken), and write the change list —
each item anchored (`upload/client.ts:88 — wrap post() in retry(3)`), plus a
proposed done-when the build can adopt verbatim ("`npm test -- upload` passes
including the new retry-exhaustion case"). The one genuine fork (user-visible
error copy) becomes `.atrium/decision.json`, not a guess.

## Finishing

Finish per the **post-results** skill: the plan as the body (proposed done-when
as its own section), file anchors and prior-step ids in `refs`. `result:
success` means the plan is executable and judgeable — not that you like it.
