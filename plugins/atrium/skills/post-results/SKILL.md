---
name: post-results
description: The step-outcome contract — how any workflow step leaves its durable record behind via .atrium/step-result.md (result status + evidence refs + artifacts list + a summary in your own words). Every step-archetype skill ends here; the harness reads this file and posts it upstream as the step's outcome (the finish_step path). Use this whenever a workflow step finishes, succeeds or not.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
---

# Post a step's results

Every workflow step — explore, plan, build, validate, whatever — leaves exactly
one durable record behind: `.atrium/step-result.md`. The harness reads this file
after your session ends and posts it upstream as the step's outcome via the
`finish_step` path (summary, result status, evidence refs, artifacts). The
pipeline routes on it — `on_success` / `on_failure` edges fire off the result you
declare here. Write it honestly or the workflow takes the wrong branch.

## The file contract

Write `.atrium/step-result.md` as YAML frontmatter + a markdown body:

```markdown
---
result: success | failure | partial
refs:
  - https://github.com/acme/app/pull/42
  - insight:1a2b3c4d
artifacts:
  - .atrium/report.md
  - .atrium/checks.json
---
One tight paragraph: what this step produced and how you know.

Then whatever structure the step's `returns` asks for — findings, the plan,
the evidence table. This body IS the summary the operator and the next step
read; write it for them, in your own words.
```

- **`result`** — `success` only when the step's `success_criteria` (from
  `brief.step.params`) are verifiably met; `failure` when they are verifiably
  not; `partial` when some are and you can name which. Never declare `success`
  on assumption — this is the `done_when` honesty rule applied to the step.
- **`refs`** — evidence pointers: PR urls, Atrium ids (`insight:…`, `signal:…`,
  `workflow:…`), doc paths. Every claim in the body should be traceable to one.
- **`artifacts`** — worktree-relative paths of files the harness should capture
  and attach to this step run (reports, check outputs, screenshots). List only
  files that exist.

## Worked example

A validate step met two of three criteria. You write `result: partial`, `refs`
pointing at the PR and the failing test's output artifact, `artifacts` listing
`.atrium/checks.json`, and a body that names the failing criterion FIRST with
its evidence, then the two that passed. The workflow's `on_failure` edge fires
and the next build step starts from your diagnosis — exactly what an honest
`partial` is for.

## The moves

1. Re-read `brief.step.params.success_criteria` and the brief's `returns` —
   they define what the body must contain and what `result` means here.
2. Verify before you declare: re-run the check, re-open the PR, re-read the
   output. The body cites what you saw, not what you intended.
3. Write the file. If the step failed or stalled, say so plainly in the body —
   a truthful `failure` with a clear diagnosis is a good outcome; a faked
   `success` poisons every step after it.
4. Then stop. Blocked-and-need-a-human is not a result status — that is the
   decision pause (`.atrium/decision.json`, per the implement-job ritual);
   write the step result only for a step that actually ran to an end.
