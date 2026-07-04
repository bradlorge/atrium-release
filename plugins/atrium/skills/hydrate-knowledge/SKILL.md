---
name: hydrate-knowledge
description: The interactive context ritual — build and refine a product's living context in Atrium (domain one-liner, feature map, POV/non-goals, glossary) by suggesting a first cut, importing what exists, exploring the codebase, and asking targeted questions until it is consistent. Writes via the gated contribute_context verb; ends by wiring keep-fresh. Use for a hydrate-knowledge job or the closing step of product setup.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, mcp__atrium__orient, mcp__atrium__search, mcp__atrium__inspect, mcp__atrium__contribute_context, mcp__atrium__describe, mcp__atrium__act
---

# Hydrate knowledge (context ritual)

You are running the **hydrate-knowledge** ritual: build the product's living
context — the canonical understanding synthesis and every future agent will
read — inside Atrium, not in git. Git is a *source by reference*; Atrium's
context object is the store. This is an interactive ritual: your questions to
the operator ARE the work — use decision pauses (`.atrium/decision.json`, per
the triage-signal ritual) rather than inventing answers. The `.atrium/`
contract and guardrails apply as usual; you reach Atrium through the compact
surface (see the pack README, "How you reach Atrium").

The target state — all four, mutually consistent:

- a **domain one-liner** (what the product is, for whom, in one sentence),
- a **feature map** (the product's real capabilities, named the way the team
  names them),
- **POV / non-goals** (what it deliberately is and is not),
- a **glossary** (the team's terms, including the surprising ones —
  "caffeine = tracked nutrient" is exactly the kind of entry that matters).

## 1 · Suggest a first cut

Call `orient()` — its `product_context` section IS the current context object
(one-liner, feature map, glossary, working notes), alongside the product
settings and module state. Skim the recent signal shape with
`search("", scope: ["insights", "signals"])` and note the repo name(s). Draft a
first-cut context from all of it. Wrong-but-concrete beats blank: the operator
corrects a draft far faster than they author from nothing.

## 2 · Ask for what exists

Ask the operator (decision pause) what already describes the product — README,
docs site, pitch deck, internal wiki, an old spec. Import what they point at:
distill each source into the four target artifacts, marking provenance in
`.atrium/PROGRESS.md`. Imported prose is a *source*, not gospel — flag where
sources contradict each other or the code.

## 3 · Offer codebase exploration

Offer to explore ALL of the product's repos (not just the one you may be
sitting in) and run a context import per repo — routes, models, feature flags,
and naming conventions reveal the real feature map and glossary candidates.
The import action lives behind the hatch: `describe("context_import")` for its
contract, then `act("context_import", { product_id, … })`. Code is evidence of
what the product *does*; the operator still owns what it is *for*.

## 4 · Refine until consistent

Iterate with targeted questions — one genuine fork at a time, concrete options
offered: "The code says 'workspaces', your README says 'projects' — which is
the canonical term?" Stop when the one-liner, feature map, POV/non-goals, and
glossary agree with each other and with what you saw. Write through the ONE
staged verb as sections settle — `contribute_context({ product_id, kind, payload })`:

- `kind: "context"` — the core artifacts (one-liner, feature map, POV/non-goals),
- `kind: "glossary_term"` — one term per call (lands as *proposed*, so the
  operator confirms),
- `kind: "working_note"` / `"note_revision"` — authored notes and their revisions.

Every write runs the same autonomy gate + audit as always. Never hold the
finished context only in scrollback.

## 5 · Keep-fresh

Fresh context decays. Before finishing: register a maintenance item so
staleness resurfaces on a cadence, and add a short doc-freshness note to the
repo's `AGENTS.md` (or its equivalent) telling future sessions that the
product context lives in Atrium and how to nudge a re-hydrate when they notice
drift. (If setup's step 4 already wired the post-merge staleness hook, point at
it rather than duplicating.)

## Worked example

`orient()` shows an empty glossary and a two-line feature map. You draft a
one-liner, the operator corrects "customers" to "clinics" (decision pause),
`act("context_import", { product_id, repo: "acme/app" })` surfaces the term
"episodes" from the models, and the operator confirms it — so you call
`contribute_context({ product_id, kind: "glossary_term", payload: { term:
"episode", definition: "one clinic visit …" } })` and
`contribute_context({ product_id, kind: "context", payload:
{ domain_one_liner: "…", feature_map: […] } })`. `orient()` now reflects all
four artifacts; you wire the keep-fresh note and finish.

## Finishing

Finish per the triage-signal ritual (`.atrium/report.md`: what was written,
what the operator confirmed, what stayed open) and — when running as a
workflow step — per the **post-results** skill, with the context/glossary refs
in `refs`. An honest gap ("non-goals unconfirmed — operator deferred") is part
of the context, not a failure to hide.
