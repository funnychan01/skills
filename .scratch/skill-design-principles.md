# Skill Design Principles

Condensed from [skill-system-reverse-engineering.md](skill-system-reverse-engineering.md) (2026-08-21), with corrections from [research-generalization-audit.md](research-generalization-audit.md) applied. Principles are harness-neutral: they describe how to encode expert judgment for agents, independent of any specific skill framework.

## Mental model

A skill system is a small, composable process system: expert judgment is encoded as gates, artifacts, and decision rights, not as prose advice. Skills are phases, not features. State lives in artifacts, not conversation. Gates are prose, not machinery. The human is a scheduled resource.

## Principles

1. **Decide who can fire it before writing a word.** Whether only a human may initiate it or an agent may reach for it on its own is the first design decision; it drives composition, documentation, and cost.
2. **Budget two loads: context load and cognitive load.** Every always-loaded line costs attention each turn; every capability only a human remembers costs human memory. Trade deliberately.
3. **Encode judgment as gates, artifacts, and decision rights.** Write the move, the stopping condition, and who decides. Skip motivational prose.
4. **Facts go to the agent, decisions to the human.** Route lookups to tools; route choices to the human with decision cost minimized.
5. **State crosses session boundaries only as artifacts.** Give each artifact a deliberate home and lifespan: durable contract, living memory, or deliberately ephemeral.
6. **End every gated step and every skill on a checkable completion criterion.** Prefer observable states ("the loop goes red") over fuzzy ones; name the rush you're defending against if the bound is fuzzy. *(Amended per audit: per-step "Done when" is a high-cost-step practice, not a universal structure.)*
7. **Protect the information hierarchy.** Inline what every branch needs; push what only some branches reach behind a pointer. Branching is the disclosure test.
8. **Compress with leading words, not sentences.** Prefer a compact pretrained term defined once over repeated restatements.
9. **Prompt the positive; reserve prohibitions for hard guardrails.** *(Aspirational: the corpus applies the hard-guardrail escape hatch wholesale. Treat as a review question, not a compliance gate.)*
10. **Degrade by protocol, never by improvisation.** Write the missing-case branch for every external dependency.
11. **Dispatch sub-agents with self-contained briefs for context isolation and AFK work.** When spawning parallel judgment-bearing sub-agents, adopt the full contract: pre-flight validation, paste-in-full briefs, evidence-only return, no-rerank aggregation. *(Split per audit.)*
12. **Keep the environment as source of truth; treat documents as caches with expiry risk.** Restate only what cannot be looked up; prune lines that stopped bearing on the task.

## Where to find the rest

- Full study (evidence, citations, workflow architecture, tensions): [skill-system-reverse-engineering.md](skill-system-reverse-engineering.md)
- Evidence-breadth audit (what survived, what was amended): [research-generalization-audit.md](research-generalization-audit.md)
- Scoring rubric for new skills: [skill-scoring-rubric.md](skill-scoring-rubric.md)
- Practical checklist and template: study sections 10 and 11
