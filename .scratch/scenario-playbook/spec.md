Status: ready-for-agent

# Spec: Scenario playbook for ask-matt (SCENARIOS.md)

## Problem Statement

When I sit down to work, I know *what situation I am in* (a hard bug, an inherited codebase, a decision blocked on a colleague, an incoming PR flood), but the skill set is indexed by *skill*, not by situation. The router `ask-matt` maps how the skills relate to each other, yet answering "I am in situation X, which skills do I run in what order, and where are the forks?" still means holding the whole flow graph in my head. The cognitive load the docs pages exist to relieve applies doubly to *combinations* of skills, which live only in prose scattered across `ask-matt`, the docs pages, and individual `SKILL.md` files.

The gap bites hardest for my in-progress EDA workflow for the ML cycle (discover unknown tables through a database MCP, infer joins, profile distributions): part of it reuses shipped skills, part of it reuses my own skills (`profiling-data`, `infer-joins`), and nothing anywhere documents how those compose or which hops are still gaps to build.

## Solution

A scenario-indexed playbook, `SCENARIOS.md`, living inside the `ask-matt` skill directory beside `PHASE-BOUNDARIES.md`. It holds thirteen scenario cards: for each situation, the entry skill, the chain, the exit condition, and every fork point inside the chain rendered as a table (condition in the left column, what to do in the right). Documented edges form the backbone; hops that no `SKILL.md` or docs page records are carried too, but flagged as proposals with their rationale. The router `ask-matt` gains a pointer to it, so the playbook is a node the router knows about rather than a competing map.

## User Stories

1. As a user with a new feature idea, I want a card for the idea-to-ship spine, so that I can confirm the canonical route (grill, spec, tickets, implement) in one glance without re-deriving it.
2. As a user facing an effort too big for one session, I want a card that shows when to reach for `wayfinder` and where its map merges back onto the main flow, so that I don't loop the map straight into implementation and lose the collapse step.
3. As a user inheriting an unfamiliar codebase, I want a card covering the entry through `research` and `triage` into the main flow, so that I have a documented route for the situation the main flow assumes away.
4. As a user chasing a hard bug or performance regression, I want a card showing `diagnosing-bugs` and its handoff to `improve-codebase-architecture` when no seam exists, so that the diagnosis doesn't dead-end when the real finding is architectural.
5. As a user doing periodic architecture upkeep, I want a card showing how `improve-codebase-architecture` generates ideas that enter the main flow at `grill-with-docs`, so that upkeep produces tracked work instead of forgotten observations.
6. As a user with a design question that won't settle on paper, I want a card for the prototype detour bridged by `handoff` in both directions, so that I know how to leave the thread and how to return to it.
7. As a user facing a flood of external issues or PRs, I want a card for `triage`, including which issues must *not* be triaged, so that I don't re-triage issues my own `to-tickets` already made agent-ready.
8. As a user blocked on knowledge that lives in someone else's head, I want a card for `to-questionnaire` showing where the answers re-enter the flow, so that the wait becomes material instead of a stall.
9. As a user mid-ticket who hits a merge conflict, I want a card showing `resolving-merge-conflicts` and the route back into the interrupted work, so that the conflict doesn't orphan the ticket.
10. As a user learning a new skill or concept, I want a card for `teach` with its prerequisites and stateful workspace, so that I know what it writes and where.
11. As a user whose conversation lost the plot, I want a card comparing `wait-what` and `handoff`, so that I pick the corrective that matches whether the context is salvageable.
12. As a user bootstrapping a new repo, I want a card for `setup-matt-pocock-skills` as the run-once precondition, so that the tracker, labels, and domain layout exist before any other engineering flow runs.
13. As a user running EDA in the ML cycle, I want a card for the extension workflow (database discovery, join inference, distribution profiling), flagged as in progress, so that I can see which hops are shipped skills, which are my own skills, and which are gaps I still need to build.
14. As a user of the EDA workflow, I want the card to show `research` as the discovery front end whose cited markdown becomes `infer-joins`' data-description input, so that the currently unrealized discovery step has a documented shape.
15. As a user of the EDA workflow, I want the card to show `domain-modeling` wired in lazily to persist confirmed grain and entity language, so that the EDA process accumulates a vocabulary instead of rediscovering one each run.
16. As a user of the EDA workflow, I want the card to mark grilling, the prototype doctrine, phase gates, and subagent parallelism as borrowed patterns rather than wired skills, so that I don't build redundant wrappers around what `profiling-data` and `infer-joins` already embody.
17. As a reader of any card, I want the fork points rendered as table rows, so that I scan for the row matching my situation instead of reading the whole card.
18. As a reader of any card, I want phase-boundary choices (continue, `/clear`, `handoff`, subagent, `/compact`) surfaced at the hops where they matter, so that the context-hygiene decision is part of the route, not an afterthought.
19. As a reader, I want every undocumented hop flagged as a proposal with its rationale, so that I can trust the backbone as quoted fact and weigh the extensions as inference.
20. As a future maintainer of the playbook, I want `ask-matt` to point at `SCENARIOS.md`, so that the router stays the single entry point and the two files re-sync together under the same trigger.
21. As a future maintainer, I want the playbook's scope pinned to the shipped buckets with a single sanctioned exception, so that in-progress and misc skills don't silently accrete into it.
22. As a reader, I want the S1 card kept deliberately short, so that the playbook doesn't duplicate what `ask-matt` and the docs pages already cover for the documented spine.

## Implementation Decisions

- **Home.** The playbook is a support file of the `ask-matt` skill, beside its existing `PHASE-BOUNDARIES.md`. It is not a published docs page: the docs tree is one page per skill, and a support file carries no URL contract. Links inside it are repo-relative.
- **Card anatomy.** Each card carries: entry skill, chain, exit condition, and the fork points inside the chain as table rows (condition left, what to do right), per the repo convention that branches go in a table or a list, never in a paragraph.
- **Depth budget.** S1 (the documented spine) gets the shortest card, reference-only. S2 through S5 (wayfinder, inherited codebase, hard bug, upkeep) get the deepest treatment, because their combination choices are the least obvious. S6 through S12 get standard treatment.
- **Grounding policy.** Documented edges form the backbone and are quotable from a `SKILL.md` or docs page. Undocumented hops (for example, `resolving-merge-conflicts` back into the interrupted work, and the EDA wiring) are carried as flagged proposals, each with its rationale.
- **EDA wiring.** Three shipped skills are wired in: `research` as the discovery front end feeding `infer-joins`' data-description input; `domain-modeling`, lazily, persisting confirmed grain and entity language; `wizard`, conditionally, only where connector provisioning is a human-only step. Grilling, the prototype doctrine, phase gates, and subagent parallelism are acknowledged as already borrowed by `profiling-data` and `infer-joins` and are deliberately not wired, to prevent redundancy.
- **Scope.** The playbook covers the 22 shipped skills in the promoted buckets only. The EDA card is the single sanctioned exception, referencing two skills that do not exist in this repo; they are named in plain text, unlinked.
- **Router re-sync.** `ask-matt`'s `SKILL.md` gains a pointer to the playbook, so the router knows the scenario-indexed view exists. The same trigger that re-syncs `ask-matt` (a skill added, renamed, removed, or repositioned in the flows) applies to the playbook.
- **Changeset.** A changeset entry records the addition, per repo convention.
- **Prose.** No em-dashes anywhere in the new prose, per repo convention.
- **Domain language.** The playbook uses the glossary terms from `CONTEXT.md`: Issue, Issue tracker, Triage role, Decision ticket. It avoids "ticket" outside Decision ticket and quotations of external systems.

## Testing Decisions

- This is a docs-only change. There is no executable code and therefore no automated tests; no test seam exists, and none is proposed.
- The seam that matters is a prose contract: `SCENARIOS.md` must not contradict `ask-matt/SKILL.md`, and the router must point at the playbook. Verification is by checklist:
  - No em-dashes in any new prose.
  - Every backbone hop is quotable from a `SKILL.md` or docs page.
  - Every undocumented hop carries a proposal flag and a rationale.
  - The EDA card carries the "in progress: extension workflow" flag.
  - `ask-matt/SKILL.md` references the playbook.
  - A changeset entry exists.
- Prior art for the shape: `ask-matt`'s own flow taxonomy and `PHASE-BOUNDARIES.md`'s ordered branching, both in the same skill directory.

## Out of Scope

- Building any new skill, including the EDA workflow's discovery front end. The playbook documents the gap; it does not fill it.
- Modifying any per-skill docs page under `docs/`.
- Changes to `README.md`, `.claude-plugin/plugin.json`, or any `SKILL.md` other than the `ask-matt` pointer.
- Upstreaming to `mattpocock/skills`. The work lands on the local `dev` branch; upstreaming is a later decision.
- Closing the ten gap findings from the exploration (the unlinked writing chain, `teach`'s missing intake interview, `code-review`'s missing standards-document producer, the orphaned misc bucket, and the rest). They are candidate material for flagged proposals in the cards and for future work, not deliverables here.
- Diagram or HTML rendering of the playbook. An explorable diagram is a possible follow-up once the prose settles.

## Further Notes

- The exploration that produced this spec inventoried 29 shipped skills (22 in the promoted buckets the playbook covers), all documented cross-skill edges, and ten notable gaps where combinations are plausible but undocumented. The gap list is the natural quarry for the playbook's flagged proposals.
- The EDA card is expected to evolve fastest: as the discovery front end gets realized, the card's flagged status should be revisited and the wired-vs-borrowed table re-checked.
- `SCENARIOS.md` and `PHASE-BOUNDARIES.md` together make `ask-matt` a three-file router: the `SKILL.md` maps flows by skill, `PHASE-BOUNDARIES.md` maps the context-hygiene decision, and `SCENARIOS.md` maps flows by situation. Keeping those three views consistent is the maintenance contract this change introduces.
