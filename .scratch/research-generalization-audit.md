# Generalization Audit: Is the Skill Design Constitution / Scoring Rubric Over-Fit to One Exemplar?

**Status: research findings, not a finished policy change.** Nothing below edits the study, the rubric, or any skill file. Amendments are proposals for the user to accept, reject, or grill.

**Date**: 2026-08-21. **Method**: repo-only primary-source audit (no web sources).
**Evidence labels**: `[stated]` = written in the repo; `[observed]` = pattern detected by grep/read; `[interpretation]`. Confidence: H/M/L per finding.

---

## 1. Research question

A reverse-engineering study (`.scratch/skill-system-reverse-engineering.md`) produced a 12-principle "Skill Design Constitution" (study §9) and a 12-axis scoring rubric (`.scratch/skill-scoring-rubric.md`). Scoring all 29 shipped skills against the rubric yielded 4/29 passing, with:

- axis 10 (Testability) = 0 for 24 of 25 failing skills,
- axis 8 (Composability) = 0 for every standalone skill,
- wrapper skills (grill-me, grill-with-docs, implement) failing by construction.

Hypothesis under test: **some principles/axes were over-generalized from ONE exemplar skill rather than a group of skills.** This audit establishes the actual support breadth of every principle and axis, using the repo as ground truth.

---

## 2. Method

1. Read the three source documents in full (study, rubric, handoff scoring report).
2. For each principle/axis, grep/read every shipped `SKILL.md` (18 engineering, 7 productivity, 4 misc = 29; `in-progress/` excluded from counts but noted where relevant) and classify support breadth: GENERAL (≥4 skills), PARTIAL (2–3), SINGLE-EXEMPLAR (1), NOT-FOUND.
3. Spot-check 17 of the study's specific citations (file + line) against the actual files (§5 below).
4. Corroborate each axis against the per-axis score distribution in the handoff report.
5. Search for in-repo testability-evidence conventions the rubric may have missed (CHANGELOG.md, `.changeset/`, `SCENARIOS.md`, CLAUDE.md maintenance rules).

Grep patterns used included: `disable-model-invocation`, `call the Skill tool`, `Done when`, `setup-matt-pocock-skills` (redirect), `ready-for-agent`, `recommended answer`, `sub-?agent`, `failure mode`, leading words (`tight|red|seam|frontier|fog of war|tracer bullet|deep module`), run-evidence terms (`ran|tested|dry-run|drift|no-op`), gate phrasing, standalone markers.

---

## 3. Per-principle audit (study §9)

Breadth counts refer to the 29 shipped skills unless noted.

| # | Principle (abridged) | Support breadth | Verified evidence | Suspected exemplar(s) | Conf |
|---|---|---|---|---|---|
| 1 | Decide invocation first | **GENERAL (29/29 stance; 14 flagged)** `[stated+observed]` | `.agents/invocation.md` L3–6; grep: 14 shipped skills carry `disable-model-invocation: true`, the other 15 are model-invoked by omission; `CLAUDE.md` L19 | none — genuinely corpus-wide | H |
| 2 | Budget context load + cognitive load | **GENERAL as trade; SINGLE as cure** | Doctrine `[stated]` `writing-for-agents/SKILL.md` L20–27; the router cure exists exactly once: ask-matt `[observed]` | Router cure: **ask-matt** | H |
| 3 | Gates/artifacts/decision rights, not advice | **GENERAL (≥8)** `[observed]` | tdd L22 seam gate; grilling L22 confirmation gate; diagnosing-bugs L66/L86; wayfinder L105 throughput cap; triage L45/L72; wizard stage confirmations; to-spec L17 seam check | none | H |
| 4 | Facts→agent, decisions→human + recommended answers | **Split: GENERAL (≥6); recommended-answer clause: PARTIAL (2–3)** | Split: grilling L20, triage L72 "Wait for direction", tdd L22–24, wayfinder L75 (HITL), wizard L18–23, to-questionnaire. Recommended answers: grilling L8/L15, setup-matt-pocock-skills L36/L55; to-questionnaire has human answer-*stubs*, a different mechanism | Recommended answers: **grilling + setup** | H |
| 5 | State crosses sessions only as artifacts | **GENERAL (≥7)** `[observed]` | wayfinder L125 resolution comments; to-tickets L63; triage AGENT-BRIEF; handoff L8 temp-dir doc; prototype L26 branch capture; to-spec L19 tracker publish; `.out-of-scope/` | none | H |
| 6 | Every step ends on a checkable+exhaustive criterion | **Doctrine stated; practice PARTIAL (3 shipped skills per-step)** | Doctrine `[stated]` writing-for-agents L47–52. Explicit "Done when:" only in to-questionnaire (3 steps), wizard (2 steps), diagnosing-bugs (1). Skill-level exits wider (grilling L22, wayfinder L84, SCENARIOS.md cards). The study itself graded this M (P9, §13 row 8) | **wizard / diagnosing-bugs / to-questionnaire** | H |
| 7 | Information hierarchy; branching disclosure test | **GENERAL (≥10 with disclosed files)** | tdd→tests.md/mocking.md; prototype→LOGIC.md/UI.md; domain-modeling→CONTEXT-FORMAT/ADR-FORMAT; codebase-design→DEEPENING/DESIGN-IT-TWICE; wizard→template.sh; triage→AGENT-BRIEF/OUT-OF-SCOPE; teach→3 format files; ask-matt→PHASE-BOUNDARIES/SCENARIOS; counterexamples (to-spec template inline, code-review smell baseline inline) follow the stated rule | none; doctrine home is writing-for-agents | H |
| 8 | Leading words, not sentences | **GENERAL (≥8 skills use the shared vocabulary)** | tight (diagnosing-bugs L20, ask-matt L44, teach L108), red (tdd, diagnosing-bugs), seam (tdd L20, to-spec L15, ask-matt L61), frontier (grilling L8, wayfinder L69), fog of war (wayfinder L82), tracer bullet (tdd L32), deep module (codebase-design, improve-codebase-architecture L13) | none | H |
| 9 | Prompt the positive; prohibitions only as hard guardrails | **Doctrine stated; corpus compliance NOT-FOUND / contrary** | Doctrine `[stated]` writing-for-agents L74. Corpus is saturated with prohibitions `[observed]`: "never runs `--abort`" (ask-matt L81), "No test is written at an unconfirmed seam" (tdd L22), wizard's negative trigger (L3), to-spec "Do NOT interview" (L7), handoff L12. The study itself logged this as tension §8.1. Only compliant text is the doctrine's own escape-hatch usage | **writing-for-agents** (doctrine); corpus *violates* it | H |
| 10 | Degrade by protocol, never improvisation | **GENERAL (≥7)** | Setup redirects: to-spec L9, to-tickets L11, code-review L13, wayfinder L25, triage L43 (5 sites, 4 verbatim `[stated]`); lazy create: domain-modeling L40; degrade: code-review L32/L72 "no spec available"; escalate: diagnosing-bugs L55; re-pitch: wait-what L7; abort: wayfinder L112 | none | H |
| 11 | Sub-agents: self-contained briefs, evidence-only, no reranking, pre-flight | **Dispatch: GENERAL (6 skills); full contract: SINGLE-EXEMPLAR** | Dispatch: code-review L11/L58, codebase-design DESIGN-IT-TWICE.md L19–23, wayfinder L115, grilling L20, improve-codebase-architecture L27, research (is one). But the *contract clauses* — pre-flight validation (code-review L23), paste-in-full briefs (L63), no-rerank aggregation (L76–78) — exist **only in code-review** | **code-review** | H |
| 12 | Environment = source of truth; docs = caches | **Doctrine SINGLE-stated; compliance GENERAL-ish (≥7)** | Doctrine `[stated]` writing-for-agents L79. Compliance: 5 skills resolve tracker config at runtime via pointers instead of restating it; tdd L10 & diagnosing-bugs L10 read CONTEXT.md "(if it exists)"; domain-modeling L64 keeps CONTEXT.md glossary-only | Doctrine: **writing-for-agents** | H |

**Principle verdicts summary**: 8 of 12 survive as corpus-wide (1, 3, 4-split, 5, 7, 8, 10, 12-compliance). Four have a narrow evidence base: **P6** (3 exemplars), **P9** (doctrine contradicted by corpus), **P11-contract** (code-review alone), **P2-router-cure** (ask-matt alone, though the underlying trade is corpus-wide).

---

## 4. Per-axis audit (rubric)

Score distributions from the handoff report (29 skills). "Zeros" = skills scoring 0 on the axis.

| Axis | Breadth of the underlying pattern | Zeros in report | Corroboration | Suspected exemplar | Verdict | Conf |
|---|---|---|---|---|---|---|
| 1. Invocation fit | GENERAL (29/29) | 0 | scores 2–3 everywhere | none | KEEP | H |
| 2. Failure mode | PARTIAL as *explicit naming* (diagnosing-bugs L66 "the exact failure this skill prevents"; writing-for-agents L43/L74); implicit in most via gates `[interpretation]` | 1 (implement) | mostly 2–3; defensible only if gate-presence counts as naming | diagnosing-bugs phrasing | KEEP, note: accept implicit naming via gates | M |
| 3. Completion criteria | Doctrine stated; per-step practice PARTIAL (3 skills) | 2 (grill-me, grill-with-docs — wrappers) | scores 1–3 scattered, matches mixed practice | wizard/diagnosing-bugs/to-questionnaire | AMEND (same fix as P6) | H |
| 4. Decision routing | GENERAL for workflow skills (6+); genuinely absent by design in reference/utility skills (writing-for-agents, research, handoff) | 6 | zeros concentrate in reference/utility skills — axis is fair but has no N/A lane | grilling | KEEP + N/A lane for reference skills | H |
| 5. Artifact discipline | GENERAL (7+) | 2 (grilling, grill-me) | grilling is deliberately stateless (conversation consumed in-session); zero is *accurate*, not a rubric bug | wayfinder/triage | KEEP | H |
| 6. Context economy | GENERAL (disclosure in 10+) | 3 (to-tickets, resolving-merge-conflicts, scaffold-exercises) | defensible; to-tickets zero is debatable (it has Process/Reference split per CHANGELOG #469) | none | KEEP | M |
| 7. Failure branches | GENERAL (7+) | 2 (grill-with-docs, migrate-to-shoehorn) | matches; wrappers inherit branches from callees — scoring them 0 is the wrapper problem again | none | KEEP + wrapper carve-out | H |
| 8. Composability | **Callers: PARTIAL (6 skills call others); standalone is a SANCTIONED design position** `[stated]` | 10 | every zero is a callee, standalone, or misc skill. ask-matt L75/L81 documents a "Standalone" architecture category; resolving-merge-conflicts is "Standalone and off every flow" `[stated]`; misc bucket is non-promoted by design (CLAUDE.md L15/L17). Callee grilling is invoked by 5 skills yet scores 0 | **main-flow skills** (to-spec/to-tickets/implement/triage) | **AMEND** — see §6 | H |
| 9. Voice & words | GENERAL (leading-word corpus dense; imperative throughout) | 0 | scores 1–3, no zeros | none | KEEP | H |
| 10. Testability | **NOT-FOUND in skill files (0/29)** | 23 | grep for run/test/drift evidence in SKILL.md files returns nothing; axis 10 derives from the study's §10 checklist item D, which is `[recommended]`, not an observed corpus pattern. The rubric was built from a checklist the corpus itself doesn't visibly satisfy in-file | **none — aspirational import**, not abstracted from any skill | **AMEND** — see §6 | H |
| 11. Honesty | SINGLE explicit marker (diagnosing-bugs L8 "Skip phases only when explicitly justified"); study's honesty evidence is largely web-doc self-disclosure (outside this audit's scope) | 0 | all skills score 1–3; the practical reading is "no skill *claims* mechanical enforcement", which is corpus-wide by default | diagnosing-bugs + web docs | AMEND wording so the default (prose gates not oversold) earns 2 without needing in-file disclosure | M |
| 12. Pruning | GENERAL (no SKILL.md exceeds 150 lines; max = teach 140; orchestration files 7–128 lines) | 0 | scores 1–3, consistent with measured line counts | none | KEEP | H |

---

## 5. Citation spot-check results

17 citations from study §4/§7/§13 verified against repo files:

| # | Study claim | Verdict |
|---|---|---|
| 1 | `.agents/invocation.md` L16 "call the Skill tool" wording rule | ✓ exact match `[stated]` |
| 2 | grilling/SKILL.md L20 "Finding facts is your job, never the user's" | ✓ exact line |
| 3 | tdd/SKILL.md L22 "No test is written at an unconfirmed seam" | ✓ exact line |
| 4 | wayfinder L105 "never resolve more than one ticket per session" | ✓ exact line |
| 5 | prototype ambiguity protocol cited at L12 | ~ content present but the ambiguity-default sentence is L17 (L12 is branch identification) — minor line drift |
| 6 | writing-for-agents L47–52 completion doctrine; L52 "checkable and exhaustive" | ✓ verbatim |
| 7 | writing-for-agents L63–72 leading words ("fast, deterministic, low-overhead → tight"; "a loop you believe in → red") | ✓ verbatim L69–70 |
| 8 | triage/AGENT-BRIEF.md path ban | ✓ L15 "Don't reference file paths: they go stale" |
| 9 | implement violates wording rule ("/tdd" bare prose) | ✓ L9 "Use /tdd where possible"; the changeset `skill-tool-invocation-terminology.md` names exactly 10 standardized skills — implement is **not** among them, so the flagship violator remains `[observed]` |
| 10 | grill-me's entire body is one Skill-tool call | ✓ L7 |
| 11 | wizard carries the corpus's only negative trigger | ✓ L3; no other description contains a negative trigger `[observed]` |
| 12 | setup "Lead each section with the recommended answer" | ✓ L36 |
| 13 | code-review sub-agent contract (isolation, pre-flight, paste-in-full, no rerank) | ✓ L11, L23, L63, L76–78 |
| 14 | argument-hint on exactly handoff/teach/claude-handoff | ✓ shipped: handoff L4, teach L5 (+ in-progress: claude-handoff, loop-me) |
| 15 | "(if it exists)" CONTEXT.md boilerplate | ✓ exactly 2 skills (tdd L10, diagnosing-bugs L10) — the study cites exactly those two, no overclaim |
| 16 | "a changeset later standardized ten skills" | ✓ changeset names 10 skills |
| 17 | CLAUDE.md router re-sync maintenance rule | ✓ CLAUDE.md L21 `[stated]` |

**Spot-check conclusion**: no citation turned out to be a fabricated or single-skill pattern masquerading as multi-skill. The study was disciplined about labeling (its own §8.10 "Frequency is not proof" and the M-grade on P9 show it knew where support was thin). The over-generalization problem is **not in the study's pattern claims** — it is in the **rubric's transmutation of mixed/aspirational patterns into 0–3 axes with a hard no-zero bar**, especially axes 3, 10, and 11, and in axis 8's treatment of sanctioned standalone design as failure.

---

## 6. Verdicts: keep / demote / amend

### Principles (study §9)

| Principle | Verdict | Rationale / proposed wording |
|---|---|---|
| P1 invocation-first | **KEEP** | 29/29, stated doctrine, harness-enforced flag |
| P2 two loads | **KEEP** (note) | The trade is corpus-wide; add: "the router cure is the ask-matt practice, to be applied when user-invoked skills multiply" |
| P3 gates not advice | **KEEP** | ≥8 skills |
| P4 facts/decisions | **KEEP + split clause** | Replace "with a recommended answer attached" with "with the decision cost minimized (recommended answer where an interview protocol exists — grilling/setup practice; one-word acceptance where config is collected — setup practice)" |
| P5 artifacts across sessions | **KEEP** | ≥7 skills |
| P6 completion criteria | **AMEND** | Proposed: "Every *gated* step and every skill ends on a completion criterion that is checkable; exhaustive bounds where the bound is observable ('the frontier is empty'), defended fuzzy bounds otherwise. Per-step 'Done when:' blocks are the wizard/diagnosing-bugs/to-questionnaire practice for high-cost steps, not a corpus-wide structure." The study itself graded this M. |
| P7 hierarchy/disclosure | **KEEP** | ≥10 skills incl. rule-following counterexamples |
| P8 leading words | **KEEP** | ≥8 skills |
| P9 prompt the positive | **DEMOTE** | Demote from "principle" to "doctrine of writing-for-agents, currently aspirational: the corpus applies the hard-guardrail escape hatch wholesale (tension §8.1). Retain as a review question ('which prohibitions could be phrased positively?'), not a compliance gate." |
| P10 degrade by protocol | **KEEP** | ≥7 skills, 4 verbatim redirect sites |
| P11 sub-agent contract | **AMEND / split** | Split into (a) "sub-agent dispatch for context isolation and AFK work" — GENERAL (code-review, codebase-design, wayfinder, grilling, improve-codebase-architecture, research) — and (b) "the full dispatch contract (pre-flight validation, paste-in-full briefs, evidence-only return, no-rerank aggregation) is **code-review's practice**; adopt it when a skill spawns parallel judgment-bearing sub-agents." |
| P12 docs as caches | **KEEP** (note) | Doctrine single-sourced in writing-for-agents, but compliance is observable in ≥7 skills (config-as-truth redirects, glossary-only CONTEXT.md) |

### Axes (rubric)

| Axis | Verdict | Proposed replacement wording (for amendments) |
|---|---|---|
| 1 Invocation fit | KEEP | — |
| 2 Failure mode | KEEP + scoring note | Add: "The failure mode may be named implicitly by the gate that blocks it (a skill whose hardest gate exists only because of a named failure counts at level 2)." |
| 3 Completion criteria | AMEND | Level 3: "Skill-level exit criterion is checkable; per-step 'Done when:' present on the steps where a wrong run is expensive; fuzzy bounds defended." Level 2: "Checkable skill-level exit." (Removes the penalty for skills that correctly skip per-step criteria.) |
| 4 Decision routing | KEEP + N/A lane | Add: "Reference/utility skills with no user decisions to route may score the axis as N/A rather than 0, provided they state what the consuming human/agent decides." |
| 5 Artifact discipline | KEEP | grilling's 0 is accurate (deliberately stateless), not a rubric bug |
| 6 Context economy | KEEP | — |
| 7 Failure branches | KEEP + wrapper carve-out | Add: "Wrapper skills inherit failure branches from the skill they delegate to; score the wrapper on whether the delegation preserves them." |
| 8 Composability | **AMEND** | Replace level 0/1 with: "**0** = composable by design but neither calls nor is called, and not documented as standalone; **1** = documented standalone (the router or its own description says so — e.g. resolving-merge-conflicts 'Standalone and off every flow', wizard's Standalone route); **2** = calls model-reachable skills correctly OR is named as a callee by ≥1 shipped skill OR emits a named handoff token; **3** = entry/exit + handoff token documented and router synced (CLAUDE.md L21 maintenance rule)." Rationale: standalone is a *stated* architecture category in ask-matt; callees like grilling (called by grill-me, grill-with-docs, triage, wayfinder, improve-codebase-architecture) currently score 0 while being the most-called skill in the corpus. |
| 9 Voice & words | KEEP | — |
| 10 Testability | **AMEND** | Replace evidence requirement: "Run evidence may live in skill files **or** repo artifacts: CHANGELOG.md entries describing behavior fixes after real runs, `.changeset/` entries, SCENARIOS.md re-syncs, throwaway branches (prototype/*, research/*). Levels: 0 = never run anywhere; 1 = run once (any artifact); 2 = run against a real case with a criterion checked; 3 = drift-checked across repeated runs (CHANGELOG rework entries count)." Rationale: see §6b below — the repo has a strong run-evidence convention, none of it in-file. |
| 11 Honesty | AMEND | Level 2 default: "Gates are prose and the skill nowhere claims mechanical enforcement." Level 3 reserved for explicit disclosure (diagnosing-bugs L8 style). Otherwise every honest skill needs a magic sentence to escape level 1. |
| 12 Pruning | KEEP | — |

### 6b. In-repo testability evidence the rubric missed (question 4 of the brief)

Yes — the repo has at least four run-evidence conventions, all outside SKILL.md files:

1. **CHANGELOG.md entries driven by observed runs** `[stated]`: wayfinder renamed because "People kept reading a wayfinder ticket as an ordinary implementation ticket" (1.2.0); HITL/AFK classification added because it "fixes students' reports of `/wayfinder` grilling _itself_" (1.1.0); grill-with-docs' bare-mention load fix was motivated by "the documented rough edge behind grill-with-docs' most-reported problem" (`.changeset/skill-tool-invocation-terminology.md`); `zoom-out` deleted because it "went unused in practice" (1.0.0). These are drift checks across repeated runs, recorded per skill.
2. **`.changeset/` pipeline** `[stated]`: 8 pending changesets, most named per skill (ask-matt-scenarios-playbook, wait-what-context-map, grilling-remove-em-dashes, domain-modeling-trigger-context-adr…) — a changesets-based release record implies per-change verification.
3. **SCENARIOS.md maintenance contract** `[stated]`: `.changeset/ask-matt-scenarios-playbook.md` — "both views re-sync under the same trigger"; CLAUDE.md L21 makes router re-sync a standing rule on every skill add/rename/remove. Every SCENARIOS card carries an "Exit condition" (14 cards verified).
4. **Dogfooding artifacts** `[observed]`: `.scratch/scenario-playbook/` (spec.md + 5 issue files) shows the repo ran its own to-spec/to-tickets flow; docs pages carry an "It's working if" section per CLAUDE.md L17.

This is exactly why ask-matt (2) and triage (2) earned nonzero axis-10 scores in the report while structurally identical evidence failed everyone else — the scorers applied the amendment inconsistently. The handoff report's flagged judgment call (option 3) is the right direction; the amendment above codifies it.

---

## 7. Pass-bar calibration analysis

Report distribution (29 skills): totals sorted = 35, 33, 33, 30, 29, 27, 27, 26, 26, 26, 26, 24, 23, 23, 23, 22, 22, 22, 21, 18, 18, 17, 16, 14, 14, 14, 13, 10, 10.

- Mean ≈ 22.1; median = 23. Category medians (from report): engineering 23, productivity 22, misc 16.
- Current bar (≥24 AND no axis at 0): **4/29 pass (14%)**. 12 skills clear the total; 8 fall solely to the no-zero rule, and for 7 of those 8 the zero is axis 10 (wayfinder, improve-codebase-architecture, to-spec, prototype, codebase-design, diagnosing-bugs, teach — teach also axis 8).
- Axis 10 alone: 23/29 skills at 0. An axis on which 79% of the reference corpus scores 0 is not measuring the corpus; it is measuring an absent artifact convention (§6b). `[interpretation]` **High confidence.**
- Axis 8: 10/29 at 0, every zero a standalone/callee/misc skill — sanctioned designs (§4).

**Verdict: the current bar is aspirational, not corpus-calibrated.** It was set by rounding up from 2/3 of 36 (24) with a no-zero veto imported from the checklist without checking that each axis's zero-state is *actually reachable-in-principle* for this corpus.

Corpus-calibrated alternatives (derived from the report distribution):

| Option | Bar | Passes | Notes |
|---|---|---|---|
| A. Median bar, no-zero kept | ≥23 + no zero | 4–5 | Still axis-10-dominated; not recommended alone |
| B. Median bar + amended axes 8/10/11 | ≥23, no zero under amended definitions | est. 14–17 | The intended combination: calibration + amendment together |
| C. Total-only at 24 | ≥24 | 12 | The report's option 2; keeps an uncalibrated total but removes the guillotine |
| D. Two-tier labeling | ≥24 = "pass"; 20–23 = "sound"; <20 = "rework" | quartiles | Matches the natural gaps in the distribution (24+, 21–23, ≤18) |

Recommendation `[interpretation]`: amend axes 8/10/11 first (their zeros are measurement artifacts), then set the bar at the corpus median (23) with the no-zero rule retained — the no-zero rule is only meaningful once zeros mean "genuinely absent" instead of "evidence lives elsewhere". Option D's tiers map cleanly onto visible gaps in the score distribution.

Wrapper skills (grill-me 10, implement 10, grill-with-docs 14) remain low under any bar. That is partly correct (they carry little substance) and partly the wrapper scoring problem (§4 axes 7/8). If wrappers stay in scope, they need the delegation-inheritance carve-out, or they should be scored as "delegation quality" (is the wrapper one correct Skill-tool call plus a correct register?) — a separate small rubric, not the 12-axis one.

---

## 8. Open questions for the user

1. **Amendment scope**: accept the axis 8/10/11 amendments as drafted, or grill them first (the handoff report already suggested `/grilling` for exactly this)?
2. **Pass bar**: pick from §7 options A–D; if B, confirm median (23) vs mean (22) as the anchor.
3. **Wrappers**: score grill-me/implement/grill-with-docs with a delegation carve-out, exclude them as "not full skills", or accept their structural failure and label them as such?
4. **Reference-skill N/A lane (axis 4)**: writing-for-agents, research, handoff have no decision routing by nature — N/A, or keep the zero as "this axis doesn't apply and that's fine"?
5. **Re-score?** With amended axes 8/10/11, the 8 "total-pass, zero-fail" skills likely move; does the user want a re-score pass, or mark the report as v1-with-known-bias?
6. **Principle 9**: demote "prompt the positive" to a review question, or keep it constitutional and accept that the corpus is non-compliant (the study's tension §8.1 position)?
7. Record the final decision as an ADR (per the handoff's own suggestion) — this audit is the context for it.

---

## 9. Source list

**Audit inputs**
- `.scratch/skill-system-reverse-engineering.md` (study; §4 P1–P13, §7 gates, §8 tensions, §9 constitution, §13 evidence matrix)
- `.scratch/skill-scoring-rubric.md` (12 axes + pass bar)
- `C:\Users\chanja13\AppData\Local\Temp\handoff-skill-scoring-rubric.md` (29-skill score table, key findings, open decision)

**Repo ground truth consulted**
- Governance: `.agents/invocation.md`, `CLAUDE.md`, `.claude-plugin/plugin.json` (25 entries; misc excluded from plugin), `.changeset/*` (8 changesets), `CHANGELOG.md`
- All 29 shipped SKILL.md files read or grepped; fully read: invocation.md, grilling, tdd, implement, code-review, wayfinder (L70–129), prototype, writing-for-agents, triage/AGENT-BRIEF.md, triage/OUT-OF-SCOPE.md (grep), wizard, grill-me, diagnosing-bugs (partial), handoff, to-spec, domain-modeling, teach (partial), ask-matt/SCENARIOS.md (grep)
- Dogfood artifacts: `.scratch/scenario-playbook/` (spec.md + issues/), `.out-of-scope/`

**Out of scope**: web documentation (aihero.dev) — this audit is repo-only; axis 11's web-doc evidence base is therefore flagged, not verified.

---

*Research findings, not policy. No file outside this one was modified.*
