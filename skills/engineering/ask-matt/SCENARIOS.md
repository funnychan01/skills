# Scenarios

The `SKILL.md` beside this file maps flows by skill. [PHASE-BOUNDARIES.md](PHASE-BOUNDARIES.md) maps the context-hygiene decision. This file maps flows by situation: you know where you are, find the route.

## How to read a card

Each card covers one situation and carries:

- **Entry skill**: what to invoke first.
- **Chain**: the route through the skills, in order.
- **Forks**: every branch inside the chain, as a table. Find the row matching your situation.
- **Exit condition**: how you know the route is done.

Two kinds of hops appear in the cards:

- **Backbone hops** are documented: each is quotable from a `SKILL.md` or docs page.
- Hops marked **(proposal)** are not documented anywhere yet. They are carried here with their rationale so they can prove out or be dropped.

## Scope

This playbook covers the shipped skills in `engineering/` and `productivity/` only. One sanctioned exception: card S13, an extension workflow in progress, which composes shipped skills with skills that do not live in this repo. Those outside skills are named in plain text, without links.

The maintenance contract: whenever a skill is added, renamed, removed, or repositioned in the flows, re-read this file together with `ask-matt`'s `SKILL.md`. The two views re-sync under the same trigger.

## S1: New feature idea, ship it

The documented spine. This card is deliberately short; the full route lives in `ask-matt`'s `SKILL.md` under the main flow, and this card only orients.

**Entry skill:** `/grill-with-docs` when you have a working directory, `/grill-me` when you don't.

**Chain:** grill, then `/to-spec`, then `/to-tickets`, then `/implement` per ticket with `/clear` between tickets. `/implement` drives `/tdd` internally and closes out with `/code-review` before committing.

**Exit condition:** every ticket implemented, reviewed, and committed.

| Condition | What to do |
| --- | --- |
| A question needs a runnable answer (state, logic, a UI you must see) | Detour: `/handoff` out, `/prototype`, `/handoff` back, reference it from the idea thread |
| The whole build fits one session | Skip `/to-spec` and `/to-tickets`; go straight to `/implement` in the same context |

## S2: The effort is too big for one session

**Entry skill:** `/wayfinder`. It needs the tracker wiring from `/setup-matt-pocock-skills`, and falls back to local markdown files without it.

**Chain:** chart the map (destination first, then decision tickets), resolve frontier tickets one at a time until the fog clears, then hand off: `/to-spec` collapses the map's linked decisions into a spec, then `/to-tickets`, then `/implement` as usual. Wayfinder plans, it does not do: a cleared map hands off, it never carries on into code.

**Exit condition:** a collapsed spec on the tracker, and implementation tickets with blocking edges. The session that finishes the map points you at a spec, not a pull request.

| Condition | What to do |
| --- | --- |
| The question can be settled by talking it through (the default ticket type) | A `grilling` ticket: resolved by `grilling` plus `domain-modeling` in a fresh session |
| "How should this look or behave": talking cannot settle it | A `prototype` ticket: the built artifact is linked from the ticket, and the selection between variants is yours to make |
| A fact outside the working directory blocks a decision | A `research` ticket: an AFK subagent burns these down in parallel on a `research/<name>` branch, the only exception to one ticket per session |
| Manual work blocks a decision (provisioning access, moving data) | A `task` ticket: it unblocks a decision, it never delivers a slice of the destination |
| A session has already grown too big mid-conversation | Say "hand off to `/wayfinder`": `handoff` bridges into a map as well as out of one |
| The effort fits one session after all | `/grill-with-docs` instead: session count, not project size, is the split |
| The deciding is already done | Skip the map entirely; go straight to `/to-spec` |

Phase boundary note: one ticket per session is the rule, so `/clear` between tickets is the default move. Scope the map to a bounded destination: maps scoped to one epic behave better than a sprawling "implement V1", and a map whose later tickets rest on assumptions the earlier ones invalidate is the waterfall trap.

## S3: Inherited or unfamiliar codebase

**Entry skill:** `/research`, then `/grill-with-docs`. **(proposal)** No `SKILL.md` or docs page documents this route as a named scenario; the hops are documented individually, and the composition is the proposal. Rationale: the main flow assumes you already understand the codebase, and nothing else covers arriving in one you don't.

**Chain:** run `/research` on the questions "what is already true here": architecture, conventions, the areas you will touch; it leaves a cited markdown file in the repo while you keep working. Take that file into `/grill-with-docs`, which sharpens your first real plan and leaves its vocabulary in `CONTEXT.md` and ADRs as it goes. Then the main flow as usual.

**Exit condition:** `CONTEXT.md` holds the language you now share with the codebase, and your first change has a tracked route into the main flow.

| Condition | What to do |
| --- | --- |
| The fog is mostly "what is already true here" and the first effort is itself multi-session | `/wayfinder` is arguably sharper here than greenfield, per its own docs; it is used routinely on legacy and half-built codebases |
| A backlog of raw issues and PRs arrived with the codebase | `/triage` first: it produces agent-ready issues that `/implement` later picks up |
| You need the vocabulary before you can plan | Let `/grill-with-docs` drive `domain-modeling`; it creates `CONTEXT.md` lazily as terms get resolved |

## S4: Hard bug or performance regression

**Entry skill:** `/diagnosing-bugs`. It is model-invoked, so it fires on "diagnose" or "debug this", or on a report that something is broken, throwing, failing, or slow.

**Chain:** build a **tight feedback loop** (one named command, already run, that goes red on this bug), then minimise the repro, rank hypotheses, instrument, fix with a regression test written before the fix, clean up. The phases are gates, not a checklist: no theory before the loop exists.

**Exit condition:** the original repro no longer reproduces, the instrumentation is gone, and the commit message names which hypothesis was right.

| Condition | What to do |
| --- | --- |
| The bug arrives as a raw report from someone else | `/triage` first: it does a shallower pass of the same first two phases and hands over verified material |
| A slow endpoint or timing regression with a known before-and-after | Same skill, performance branch: measure a baseline, then bisect; measure first, fix second |
| No correct seam exists for the regression test | The absence is itself the finding: the post-mortem hands off to `/improve-codebase-architecture`, after the fix is in |
| "Where are the bottlenecks?", no specific symptom | No skill covers this today; the proactive-audit skill was proposed and closed. Diagnose one named failure instead |
| The loop cannot be built at all | The skill stops, lists what it tried, and asks you for access, an artifact, or permission to instrument; it does not hypothesise anyway |

Phase boundary note: the recommendation to run `/improve-codebase-architecture` lands after the fix is committed, when there is more information: that is a natural boundary, and a good place for `/clear` or `/compact` before the upkeep work starts.

## S5: Periodic architecture upkeep

**Entry skill:** `/improve-codebase-architecture`, every few days or whenever you have a spare moment.

**Chain:** it scans the codebase for **deepening opportunities** and presents them as a visual HTML report, speaking the `/codebase-design` glossary throughout. Picking one generates an idea; the skill calls the Skill tool with `grilling` to walk the decision tree with you, and the idea then enters the main flow at `/grill-with-docs`.

**Exit condition:** a picked deepening opportunity has become a tracked idea in the main flow, or the scan found nothing worth doing and you close it with a clear conscience.

| Condition | What to do |
| --- | --- |
| You picked a deepening opportunity | Take it to `/grill-with-docs`, then the main flow: the survey finds candidates, `/codebase-design` is the bench the chosen one gets designed on |
| A term in the report isn't in the `/codebase-design` glossary | Reach for one that is; the report's language stays inside that vocabulary |
| Nothing worth doing this pass | Close it. Upkeep is periodic by design; a clean scan is a result, not a failure |
| A hard bug just revealed there is no good seam | You arrived here from `/diagnosing-bugs`' post-mortem: the finding names where to scan |

Phase boundary note: upkeep sessions are short and self-contained, so `/clear` after each scan is the cheap default; carry only the picked idea forward, via the main flow's own context rather than a summary.

## S6: A design question that won't settle on paper

**Entry skill:** `/prototype`, bridged by `/handoff` when the question lives inside a larger thread.

**Chain:** `/handoff` out, `/prototype` answers the question with throwaway code (does this state model feel right, what should this UI look like), `/handoff` back what you learned, reference it from the original thread. Throwaway is a constraint on how the code is written, not a promise to destroy it: the answer folds into the real code, and the prototype stays as a primary source on a `prototype/<name>` branch.

**Exit condition:** the design question has a runnable answer, pointed at from the implementation issue.

| Condition | What to do |
| --- | --- |
| The question is inside a larger grilling thread | The full detour: `/handoff` out and back, so the thread keeps its context |
| The question stands alone | `/prototype` directly, any time a design question is hard to settle on paper |

## S7: External issues or PRs piling up

**Entry skill:** `/triage`, the on-ramp for work you didn't create. It needs the tracker wiring from `/setup-matt-pocock-skills`.

**Chain:** issues move through a state machine of triage roles (categorise, verify, grill if needed via `grilling` plus `domain-modeling`, updating `CONTEXT.md` and ADRs inline) until each lands with a Triage role and an agent-ready brief. `/implement` picks the briefs up later.

**Exit condition:** every incoming item carries a Triage role, and the agent-ready ones carry briefs in the agreed format.

| Condition | What to do |
| --- | --- |
| Issues you created, or tickets `/to-tickets` produced | Don't triage them: they are already agent-ready |
| A raw bug report turns out to be a hard one | The brief hands it to `/diagnosing-bugs`, which does the thorough version of triage's first two phases |

## S8: Blocked on someone else's knowledge

**Entry skill:** `/to-questionnaire`. It is the inverse of `/grill-me`: it interviews you about the send (who it goes to, what you need back) and aims the questions at the gap.

**Chain:** the questionnaire goes out; what comes back is material for `/grill-with-docs` or `/to-spec`, where it rejoins the main flow.

**Exit condition:** the returned answers have re-entered a grilling or spec conversation, and the decision they were blocking is now answerable.

## S9: Mid-ticket merge conflict

**Entry skill:** `/resolving-merge-conflicts`, standalone and off every flow: reach for it when you are already mid-conflict.

**Chain:** it works the in-progress merge or rebase hunk by hunk, resolving each by **intent** traced to each side's primary source rather than by picking lines, then finishes the operation. It never runs `--abort`. **(proposal)** The return hop, resuming the interrupted `/implement` or `/tdd` work after the conflict clears, is documented nowhere; the rationale is that this is the most common real-world entry point into the skill, and the route back is the half that's missing.

**Exit condition:** the merge or rebase completes, and the interrupted ticket's work resumes on the resolved tree.

| Condition | What to do |
| --- | --- |
| You are mid-ticket in `/implement` or `/tdd` | Resolve the conflict, then resume the same ticket: the regression tests you already have are your check that the resolution held |
| The conflict arrived outside any flow | Resolve it, then `/clear`: nothing from the conflict session matters to what's next |

## S10: Learning a concept in this repo

**Entry skill:** `/teach`. It uses the current directory as a stateful workspace over multiple sessions.

**Chain:** teach builds its workspace (mission and learning records) in the directory and drives the learning across sessions.

**Exit condition:** the learning goal is met or explicitly parked, with the record of where it stopped left in the workspace.

## S11: The conversation lost the plot

**Entry skill:** `/wait-what` if the context is salvageable, `/handoff` if it must travel.

**Chain:** `/wait-what` works after the fact, inside any other skill: the agent re-pitches the message that didn't land, in plain English, using the `CONTEXT.md` vocabulary. `/handoff` instead when the work is travelling to a new harness, a new directory, or a colleague.

**Exit condition:** the misread is corrected and the work continues, or the work has a portable file and a fresh session to continue in.

| Condition | What to do |
| --- | --- |
| One message didn't land, the session is otherwise sound | `/wait-what`: cheap, immediate, stays in place |
| Jargon keeps arriving, not just one message | The upfront cure is `/grill-with-docs`: a shared language agreed early stops it arriving at all |
| The work must move harness, directory, or person | `/handoff`: portability is the only thing it is for |

## S12: Bootstrapping a new repo

**Entry skill:** `/setup-matt-pocock-skills`, run once before the first engineering flow.

**Chain:** it configures the Issue tracker, the triage label vocabulary, and the domain doc layout, writing `docs/agents/` and the `## Agent skills` block that the other skills resolve those through.

**Exit condition:** `triage`, `to-spec`, `to-tickets`, and `wayfinder` can all read their configuration, and `domain-modeling` knows where to create `CONTEXT.md` lazily.

| Condition | What to do |
| --- | --- |
| A custom tracker (Jira, Linear, self-hosted) | Describe the workflow in one paragraph; setup records it as prose and the skills follow it |
| No remote at all | Local markdown under `.scratch/`: the out-of-the-box fallback |

## S13: EDA in the ML cycle (in progress: extension workflow)

This card is the sanctioned scope exception: it composes shipped skills with two skills that do not live in this repo, `profiling-data` and `infer-joins`, named here in plain text without links. Both are written in this repo's house style: user-invoked, gated phases, checkpoint completion criteria, human-owned verdicts.

**Entry skill:** `/research`, aimed at the database.

**Chain (proposal):** the whole route is the proposal; no `SKILL.md` documents it. Rationale: the EDA front end of the ML cycle has no route anywhere in the set, and this composition is the shape the missing route should take.

1. `/research` explores the unknown database through its MCP connector against primary sources (schemas, row counts, content samples) and leaves a cited markdown file describing what exists. That file becomes the data-description input of the next step.
2. `infer-joins` (outside skill) infers and validates join paths across the discovered tables: govern, explore, sanitize, match, rank, validate, flag, confirm. The human owns the connector and every confirmation.
3. `profiling-data` (outside skill) runs the EDA itself: fingerprint the source, close the manifest, profile every column, answer every question, report with a requested verdict.
4. `domain-modeling`, lazily throughout: as tables, grain, and entity language get confirmed, they are recorded in `CONTEXT.md` so the next EDA run starts with a vocabulary instead of rediscovering one.

**Exit condition:** a profiling report with an explicit verdict on what happens with the dataset next, confirmed joins as parameterized SQL, and the confirmed language persisted in `CONTEXT.md`.

**Wired versus borrowed.** The split is deliberate, to prevent redundant wrappers around what the outside skills already embody:

| Matt's skill | Status here | Why |
| --- | --- | --- |
| `research` | Wired | The discovery front end: its cited markdown is exactly `infer-joins`' data-description input |
| `domain-modeling` | Wired, lazily | Persists confirmed grain and entity language; nothing in the outside skills does this today |
| `wizard` | Wired, conditionally | Only if connector provisioning is a human-only step in your environment |
| `grilling` | Borrowed, not wired | `profiling-data`'s question-driven phase 4 already asks the questions before computing anything |
| `prototype` doctrine | Borrowed, not wired | `profiling-data`'s scripts are already throwaway, one command, full state printed |
| Phase gates | Borrowed, not wired | Both outside skills already checkpoint every phase against completion criteria |
| Subagent parallelism | Borrowed, not wired | `infer-joins` already dispatches one subagent per table and per candidate join |

**Gaps to build:**

| Gap | What would close it |
| --- | --- |
| The database discovery step runs ad hoc today | A dedicated discovery skill, or a documented `/research` brief template aimed at MCP-backed databases |
| Confirmed joins and profiles don't flow into model-building work | A handoff convention from the profiling verdict into the main flow at `/grill-with-docs`, mirroring S4's post-mortem handoff |

