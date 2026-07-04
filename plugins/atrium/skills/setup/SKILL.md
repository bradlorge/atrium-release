---
name: setup
description: The product onboarding ritual — a skill-kind code job run once per repo that wires a product into Atrium's loop: discover the team's real flow FIRST (interview, never impose), then feedback ingest, a screenshot harness, release-tracking CI, doc-freshness hooks, the product's own runner skills, surfaces, and module activation. All PRs stay OPEN; every integration is proposed from what the repo already does.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, mcp__atrium__orient, mcp__atrium__search, mcp__atrium__inspect, mcp__atrium__catalog, mcp__atrium__report, mcp__atrium__describe, mcp__atrium__act
---

# Set up a product (onboarding ritual)

You are running the **setup** ritual for one repo of a product: wire it into
Atrium's loop — feedback in, releases tracked, docs fresh, its own runner
skills authored. This is a code job: the implement-job ritual applies in full
(worktree, guardrails, `.atrium/` contract, decision pauses). What this skill
adds is the onboarding sequence.

The cardinal rule: **discover, then propose — never impose.** Every integration
you add must be shaped from what this repo and this team already do. A template
pasted over a real flow is worse than nothing; it will be deleted and Atrium
will be distrusted with it.

**How you reach Atrium:** the compact surface (see the pack README). Setup is
the hatch-heavy ritual — most provisioning actions (keys, surfaces, modules)
are deliberately unadvertised admin actions, reached as
`describe("<action>")` for the contract, then `act("<action>", { … })`. A
misspelled or misshapen `act` call returns the nearest candidate actions with
their schemas — recover from the candidates rather than guessing again. Every
`act` runs the same autonomy gate + audit as an advertised tool.

## 0 · Discover & interview FIRST

Before writing anything, read the repo's actual operating reality:

- CI: `.github/workflows/*`, other pipeline configs — what builds, tests, and
  releases today, on which triggers.
- Scripts: build/test/release entries in `package.json` / `Makefile` /
  `fastlane` / whatever the stack uses.
- Branch model: default branch, protection hints, release branches/tags.
- Any existing screenshot, preview-deploy, or store-submission setup.

Also call `orient()` once — it tells you which modules are already enabled and
what context the product already holds, so you propose from Atrium's reality
too. Then confirm with the operator via decision points
(`.atrium/decision.json`, per the implement-job ritual) — one focused question
per genuine fork: "Your CI releases on tag push — is that the real flow, or do
you cut releases some other way?" Batch what you can; never guess on how a
team ships. Record the confirmed flow in `.atrium/PROGRESS.md` — every later
step is shaped by it.

## 1 · Feedback ingest

Create a publishable key and fetch the embed snippet via the hatch:
`act("listen_create_publishable_key", { product_id, … })`, then
`act("listen_get_embed_snippet", { … })` (run `describe` on each first for the
exact args). Place the snippet where this stack actually mounts such things
(root layout, app shell, template partial) — as a PR change,
commented-or-flagged if the operator should choose the mount point.

## 2 · Before/after screenshot harness

Suggest a harness FROM the repo's stack — never a generic one:

- iOS/macOS: XCTest UI tests + `simctl` screenshot capture.
- Web: Playwright with a small before/after script over the key screens.
- Other stacks: whatever the ecosystem's native capture is.

Wire it so a runner job can produce before/after evidence for a change. Reuse
any existing screenshot/E2E setup rather than adding a parallel one.

## 3 · Release-tracking CI

Atrium fires the outbound dispatch but needs the repo to report state back.
Add the submission-state callback to the repo's release pipeline — the exact
contract:

- Mint this product's signing key via
  `act("release_create_signing_key", { product_id, … })` (the secret is
  returned ONCE — store it immediately as the repo's CI secret, e.g.
  `ATRIUM_SUBMISSION_SIGNING_KEY`; lost = revoke + re-mint. Rotation and
  listing are hatch actions too: `release_revoke_signing_key`,
  `release_list_signing_keys`).
- `POST {SUPABASE_URL}/functions/v1/submission-state`
- Header `X-Atrium-Signature`: hex HMAC-SHA256 of the raw body, keyed by that
  per-product signing key.
- Body: `{ product_id, external_ref | submission_id, state, build_number, detail }`

The `submit_to_store` verb's result returns a ready-to-adapt CI template —
fetch it rather than hand-rolling. Attach the callback to the pipeline stages
the team confirmed in step 0 (e.g. TestFlight upload → `waiting_for_review` →
review outcome), so Atrium is no longer blind after dispatch.

## 4 · Doc-freshness hooks

Add a NON-BLOCKING post-merge check (CI step or hook, per the repo's
conventions) that flags likely-stale docs (README/AGENTS/docs touched-code
overlap) and, on staleness, raises an Atrium maintenance item or queues an
optional hydrate-knowledge job. It must never block a merge or a release.

## 5 · The product's own runner skills

Author `.claude/skills/*` in THIS repo, tailored to its stack — the skills a
runner session will load when working here. The proven pattern: read the
Atrium feedback for the area (the gather-feedback moves: `search` scoped to
the area, `inspect` the insights that matter) → work in an isolated worktree →
build → run the step-2 harness for before/after screenshots → open a PR —
**never merge**. Write them in this plugin's voice: frontmatter (name,
description) + a calm, imperative operating ritual, citing the repo's real
commands from step 0.

## 6 · Define surfaces

A **surface** is an independently released and managed interface of the
product — a web app, a phone/desktop app, a public API, an MCP server. Each has
its own distribution reality (continuously deployed web ≈ zero skew;
store-distributed native = high skew; versioned API = pinned skew), and
Atrium's release loop adapts per surface. Check what already exists with
`catalog("surfaces")`; from what you learned in step 0, register each missing
one via `act("surface_create", { product_id, name, type, distribution, … })`
(`describe("surface_create")` for the full schema). Confirm the list with the
operator if it is not obvious.

## 7 · Module activation + knowledge hydration

Advance the product's module state to reflect what is now wired —
`act("module_advance", { product_id, module, … })`, with `catalog("modules")`
/ `report("readiness")` to see current state — then hand off to the
**hydrate-knowledge** ritual (queue it as a follow-on job or note it in the
report) so the product's living context gets built next.

## Worked example (the hatch idiom)

You need the embed snippet but forget the exact action name:
`act("listen_embed_snippet", {})` fails soft, returning five candidates
including `listen_get_embed_snippet` with its schema. You call
`describe("listen_get_embed_snippet")`, see it wants `{ product_id, key_id }`,
mint the key first with `act("listen_create_publishable_key", { product_id })`,
then fetch the snippet with the returned key id. Both calls appear in the
audit trail under their inner action names.

## Finishing

All repo changes land as commits on the job branch — the harness opens the PR;
it stays OPEN for human review, never merged by you. Finish per the
implement-job ritual (`.atrium/report.md`: what was wired, what the operator
confirmed, what was deferred) and — when running as a workflow step — per the
**post-results** skill, with the PR, key/surface/module refs in `refs`.
