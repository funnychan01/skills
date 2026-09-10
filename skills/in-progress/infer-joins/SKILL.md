---
name: infer-joins
description: Infer join paths between database tables — explore, sanitize keys, rank candidates, validate, confirm. Run by hand.
disable-model-invocation: true
---

# Infer Joins

Infer and validate join paths across a set of database tables. The skill owns the exploration and the verification; the human owns the connector, the confirmations, and the final call. Every phase ends on a checkpoint against its completion criterion — advance on yes; on no, fix and re-run the phase.

## Inputs

- **Table names** — the tables to join, from the user.
- **DB connector** — connection the agent can run queries through. Ask for it up front and wait; nothing runs without it.
- **Data description** (if the user has one) — column glossary, known relationships. Read it before forming hypotheses.

## Ground rules

- **Govern every query.** Every query passes the *governor* (step 1) first. Tables in the millions of rows get `APPROX_COUNT_DISTINCT`, sampling, or row-limited probes instead of full scans. Log each exploration query's runtime; these feed the performance section of the report.
- **Aggregate results only.** Tables stay in the database; only counts, profiles, samples, and statistics cross into the conversation.
- **Subagents explore in parallel.** One subagent per table in step 2, one per candidate join in step 5; the main thread only ranks, validates, and talks to the user.
- **Explain the remainder.** Every unresolvable remainder — a subpopulation a candidate fails on — costs at least one probe (profile its other columns and discriminator values) before it may become a caveat or a dismissal. Remainders leave as hypotheses or open questions, never as footnotes.

## The process

### 1. Govern

Estimate size and cost before exploring: row counts (approximate is fine), and a timeout/cost bound for the exploration queries that follow.

Completion criterion:

- [ ] Every input table has a row-count estimate
- [ ] A timeout and cost bound is set for exploration queries

### 2. Explore

Dispatch one subagent per table. Each returns: schema, row count, primary-key candidates, value profiles of plausible key columns, and a short content description. Where a discriminator column exists, null rates and key profiles are reported conditional on it (type × column), not only marginal — that contingency view is what exposes which foreign keys are active for which type.

Completion criterion:

- [ ] Every input table has a profile card: schema, row count, key candidates, sample values, content summary
- [ ] Where a discriminator column exists, null rates are reported conditional on it

### 3. Sanitize

Run *hygiene* on every plausible join-key column and report what it finds:

- whitespace and padding
- case mismatches (`'NY'` vs `'ny'`)
- format drift (dates, phone numbers, ZIP codes)
- outliers in key values

Normalize only what a later join depends on; the deliverable is the report.

Completion criterion:

- [ ] Every candidate key column has a hygiene report
- [ ] Every normalization applied is listed with its affected row count

### 4. Match

Form candidate joins across the tables: name-based hypotheses, then evidence from key overlap, uniqueness, and containment. Detect composite-key candidates and non-equi patterns from column names and value distributions. Score each candidate with a stated confidence in [0, 1] and its evidence. If no candidate clears 0.3, surface the best near-misses and ask the user for direction before continuing.

Completion criterion:

- [ ] Every plausible column pair across the input tables is scored or explicitly dismissed
- [ ] Each surviving candidate carries a confidence score and the evidence behind it

### 5. Rank and route

Disambiguate the survivors: rank single-hop candidates on evidence; build the inferred relationship graph (nodes = tables, edges = candidates) and enumerate join chains with graph pathfinding, shortest paths first. Present the chains as options.

Completion criterion:

- [ ] Candidates are ranked with the top pick named
- [ ] Every multi-hop chain between requested tables is enumerated with its full path

### 6. Validate

For each top candidate, run the trial join and validate before presenting:

- **round-trip** — the source tables reconstruct from the joined result via grouping
- **orphans** — no referential-integrity violations
- **row-count arithmetic** — result size matches the expectation for the join type; cartesian products are caught here
- duplicates introduced, nulls in key columns

When a check fails on a subpopulation — orphans concentrated under one discriminator value, in particular — run the remainder probe per the ground rule before labeling it; a convenience pointer that mirrors the primary key for most types but breaks for one is a lead worth following.

Completion criterion:

- [ ] Every top candidate has all four checks run, with pass/fail and the numbers behind each
- [ ] Every failing subpopulation has at least one remainder probe recorded behind it

### 7. Flag

Compile a red/green flag dashboard from step 6 — every red flag paired with its cause and a proposed fix — and present it to the user. No confirmation request lands without its dashboard.

Completion criterion:

- [ ] Dashboard presented: green checks, red flags with cause and proposed fix
- [ ] User has seen the flags for every candidate being proposed

### 8. Confirm

Present each recommended join: the copy-paste SQL, a few result rows, and a plain-English explanation of *why* this join makes sense given the data. Open questions — unresolved remainders, missing physical tables — are presented alongside the recommendations, as questions for the user. Wait for confirmation. If the user pushes back, refine — adjust the candidate set or the join definition and re-run from the earliest affected step — then re-present. The loop closes only on explicit confirmation.

Completion criterion:

- [ ] Confirmed joins have user sign-off recorded
- [ ] Every rejection carries the user's stated reason
- [ ] Every unresolved remainder is surfaced as an open question in the presentation

## Artifacts

- **Schema graph** — the inferred relationship graph (nodes = tables, edges = relationships with confidence) as JSON.
- **Parameterized SQL** — one script per confirmed join, with `{placeholders}` for date ranges, filters, and column selections.
- **Report** — one summary per process step, dashboard-style with collapsible sections:
  - executive summary: top 3 join recommendations
  - detailed exploration: expandable per table
  - performance: per-query runtimes under the governor
  - raw logs: hidden by default

## Key-matching heuristics

Flat reference consulted in step 4, ordered by how strong the evidence is:

1. **Name match** — identical or synonym column names (`cust_id` / `customer_id`), and discriminator *values* that compose other entity names — such a value implies composition edges to those entities' tables. Hypothesis only; name matches with low value overlap are dismissed.
2. **Value overlap** — distinct-value intersection between two key candidates, measured with sampling on large tables. Overlap near 1.0 on the child side is the strongest single signal.
3. **Uniqueness and containment** — a parent key is (near-)unique; the child's values are contained in the parent's. The asymmetry names the join direction.
4. **Composite keys** — no single column matches, but a column tuple does; suspect it when names pair up (`order_id` + `line_no`) or single-column overlap stays stubbornly partial.
5. **Non-equi joins** — range or inequality relations (`BETWEEN` on dates, `>=`/`<` on intervals); suspect them from interval-shaped columns like `valid_from` / `valid_to`.
6. **Code-vs-name pairs** — one side holds a code, the other its expansion (`'HKG'` vs `'Hong Kong'`); needs a lookup table, name it explicitly in the candidate.
7. **Sibling foreign keys** — a table with two or more FK-shaped columns plus a discriminator: the type × FK-non-null contingency profile from step 2 shows which sibling keys are active per type. A type whose expected key doesn't resolve but whose siblings do gets a composition edge, not a dismissal.

Confidence is the weighted agreement of these signals; state which signals fired when scoring.
