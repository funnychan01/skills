---
"mattpocock-skills": patch
---

Add `SCENARIOS.md` to the `ask-matt` skill: a scenario-indexed view of the flows, beside the existing skill-indexed `SKILL.md` and the context-hygiene `PHASE-BOUNDARIES.md`.

- Thirteen scenario cards (S1 through S13), each with entry skill, chain, fork tables, and exit condition. S2 through S5 get deep treatment; S1 is a short reference to the documented spine.
- Documented edges form the backbone; hops not documented in any `SKILL.md` or docs page are carried as flagged proposals with their rationale.
- S13 is the sanctioned scope exception: an in-progress EDA extension workflow composing shipped skills (`research`, `domain-modeling`, conditionally `wizard`) with skills outside this repo (`profiling-data`, `infer-joins`, named in plain text).
- `ask-matt`'s `SKILL.md` gains a pointer to the new file; both views re-sync under the same trigger.
