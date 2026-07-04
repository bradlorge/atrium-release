# Atrium skills pack

**A Claude Code plugin** · version `0.2.0` · Apache-2.0

The operating rituals an agent session loads when working for Atrium — as a
Claude Code plugin, bundled with the runner and installable for any developer's
own session via the `bradlorge/atrium-release` marketplace. Each skill is a `SKILL.md`
under `skills/<name>/`; the descriptions in their frontmatter decide when they
load, and their `allowed-tools` scope each to exactly the compact-surface tools
it needs (a read-only ritual cannot write). This pack is NOT the security floor
(that ships non-pluggable via `--append-system-prompt`) and NOT the MCP server
(that ships via `--mcp-config`).

## Install

The pack is a self-contained Claude Code plugin — a `.claude-plugin/plugin.json`
manifest plus the `skills/` tree — so any developer can install it into their
own `claude` session, independent of the Atrium runner.

**Via the plugin marketplace** (recommended — versioned + pinnable):

```
/plugin marketplace add bradlorge/atrium-release
/plugin install atrium@atrium
```

The marketplace lives in the public `bradlorge/atrium-release` repo; its
root `.claude-plugin/marketplace.json` registers the single `atrium` plugin
(this directory, `plugins/atrium`), and `/plugin install atrium@atrium` installs
the pack. Pin a version with `atrium@atrium@0.2.0`.

**Directly, for a one-off session** (from a clone of this repo):

```
claude --plugin-dir ./plugins/atrium
```

(This mirrors how the runner loads it — see `runner/src/main/executor/plugin.ts`
in the source repo, which points at `runner/resources/atrium-plugin`.)

## What this needs to actually run

The skills are *rituals over Atrium's compact MCP surface* — they assume that
surface is reachable. Two things must be wired, and neither ships in this plugin
by design:

1. **The Atrium MCP server**, passed to `claude` via `--mcp-config` +
   `--strict-mcp-config`, exposed under the server name `atrium` (so its tools
   appear as `mcp__atrium__orient`, `mcp__atrium__queue`, …). You need a token
   scoped to a product. See the "How you reach Atrium" section below for the
   tool vocabulary.
2. **The untrusted-is-DATA security floor**, appended out-of-band via
   `--append-system-prompt`. It stays non-pluggable on purpose — a plugin must
   never be able to relax it.

Without the MCP server the file-contract rituals (the `.atrium/` discipline,
the durable pause) still work; the Atrium reads/verbs simply have nothing to
talk to.

## Quickstart (outside developer)

1. Install the pack (either method above — marketplace or `--plugin-dir`).
2. Start `claude` with your Atrium MCP config:
   `claude --plugin-dir ./plugins/atrium --mcp-config atrium.json --strict-mcp-config`
   where `atrium.json` names an `atrium` HTTP MCP server with your product-scoped
   Bearer token.
3. Ask the session to *"orient, then clear the triage queue"* — it loads the
   `triage-signal` ritual (scoped to the reads + triage verbs), calls
   `orient()` / `queue()`, and works the surface. No repo required for
   triage/discovery jobs; code jobs run in an isolated git worktree.

Every skill's `allowed-tools` is the exact list it may call — a read-only skill
(`explore`, `gather-feedback`, `pick-workflow`) carries no write verb, so an
unattended session cannot wander into a send while it is meant to be reading.
The server-side autonomy gate remains the real authority; `allowed-tools` is
defense-in-depth at the skill layer.

## How you reach Atrium (the compact surface)

Every skill in this pack talks to Atrium through the **compact MCP surface** —
a small set of task-shaped tools in front of the full internal registry. Three
idioms cover everything; skills below reference this section rather than
restating it.

**1 · Composed reads — discovery.** Eight read tools return assembled
documents, not row sets. Every entity mention anywhere in a document carries a
typed `ref` (`insight:<uuid>`, `ticket:<uuid>`, `work_item:<uuid>`, …) that
feeds `inspect`.

| read | what it answers |
|---|---|
| `orient()` | Session boot: account/brands/products, your roles, the product's living context + glossary + working notes, enabled modules, queue headcounts. Call it first. |
| `queue()` | Everything awaiting a decision — approvals, triage, blocked steps, unverified resolutions, outbox, KB gaps, DSRs, access requests. Each item names the verb that resolves it (`decide_with`). |
| `search(query, scope?)` | Cross-entity search (embedding-backed); an EMPTY query + scope browses most-recent-first — this replaces the old per-entity list tools. |
| `inspect(ref)` | One entity WITH its graph neighborhood — an insight with claims, evidence verbatims, linked work, PRs, releases; a ticket with its thread; a work item with its run and steps. The read that kills get-then-get-then-get chains. |
| `catalog(kind)` | The definitional vocabulary: `workflows` (pass `query` for a suggest match), `surfaces`, `lanes`, `rings`, `channels`, `modules`, `skills`, `kb_structure`, … |
| `report(topic)` | Computed rollups: `insights`, `support` (cause clusters), `surveys`, `readiness`, `usage`, `ledger`, `locale`, `competitors`, `trust`. |
| `people(query)` | The identity/audience plane: people with identities, consents, contributor status. |
| `timeline(scope?)` | What happened: the audit trail, decisions, adjudications. |

**2 · Task verbs — mutations.** The hot writes are first-class typed tools:
`decide_triage`, `update_insight`, `merge_insights`, `move_signal`,
`capture_feedback`, `add_work`, `start_step` / `finish_step`, `comment`,
`create_ticket` / `send_reply`, `contribute_context`, `verify_resolution`,
`approve` / `reject`, and their kin. Every verb runs through the same
server-side autonomy gate + audit as always — a verb is a doorway, never a
bypass. Approve-level actions stage for a human; an agent can never
self-approve.

**3 · `describe` + `act` — the escape hatch.** Rare admin/setup/diagnostic
actions are not advertised but stay fully reachable:
`describe(topic_or_action)` returns the exact schema, autonomy behaviour, and a
worked example for any registry action (or the five nearest matches for a
guess); `act(action, args)` invokes it through the SAME gate and audit as a
direct call. **The catch:** an unknown name or misshapen args never dead-ends —
`act` returns the nearest candidate actions with their schemas, so a miss is a
one-round-trip recovery. Idiom: `describe` first, then `act`.

## The skills, by job

**Job-level rituals** (the seed prompt names one):

- **implement-job** — the build/verify ritual: run a code job unattended in an
  isolated worktree; guardrails, session-start recovery, the durable
  `.atrium/` file contract, decision pauses.
- **triage-signal** — the repo-less ritual: work Atrium's queues through the
  MCP surface (triage, cluster, propose insights, draft, investigate).
- **invoke-skill** — the workflow-step wrapper: run ONE step of a pipeline
  whose brief names a step skill; the step params contract +
  `.atrium/step-result.md`.
- **setup** — product onboarding, once per repo: discover the team's real flow
  first, then wire feedback ingest, screenshots, release-tracking CI,
  doc-freshness, the product's own skills, surfaces, and modules.
- **hydrate-knowledge** — build the product's living context (one-liner,
  feature map, POV/non-goals, glossary) interactively with the operator.

**Step archetypes** (`brief.step.skillRef` names one; all end via
post-results):

- **explore** — read-only reconnaissance ending in an evidence report.
- **plan** — design the approach + propose the done-when; no production code.
- **build** — implement the step's change in the worktree; commit, never merge.
- **validate** — adversarial check of the change against `success_criteria`.
- **finalise** — tidy the branch, stage the PR material, write the hand-off.
- **gather-feedback** — read the customer signal for an area and distill it.
- **pick-workflow** — recommend which workflow graph a work item should run.

**The closing contract:**

- **post-results** — how every step leaves its durable record
  (`.atrium/step-result.md`: result + refs + artifacts + an honest summary).

## The `.atrium/` file contract (shared by all)

| file | role |
|---|---|
| `.atrium/done-when.json` | The acceptance checklist; an item flips `met: true` ONLY after verification. The harness gates completion on it. |
| `.atrium/PROGRESS.md` | Append-only notes so a resumed context window recovers state. |
| `.atrium/decision.json` | The durable pause: `{ "question", "options" }` — write it and stop instead of guessing. |
| `.atrium/checks.json` | Structured check outputs, captured as an artifact. |
| `.atrium/report.md` | The authored, human-voice run report. |
| `.atrium/step-result.md` | Workflow steps only: the outcome the pipeline routes on. |
