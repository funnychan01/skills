# Reverse-Engineering the Skill System: Mindset, Architecture, and Reusable Principles

**Sources**: repository `e:\projects\skills` (mattpocock/skills, plugin version 1.2.3) + web documentation at https://www.aihero.dev/skills (fetched 2026-08-20).
**Method**: four parallel analyst sub-agents (Repository, Workflow, Skill Design, Documentation) returned evidence; a Critical Reviewer challenged the consolidated conclusions; the lead agent verified key claims by direct reading and performed the final synthesis.
**Evidence labels used throughout**: `[stated]` = explicitly written in a source; `[observed]` = pattern detected across sources; `[recommended]` = what you should adopt. Confidence: **H**igh / **M**edium / **L**ow per conclusion. Per the agreed conflict rule, where docs and repo disagree, the repo wins as implementation evidence and the disagreement is reported.

> **Condensed version**: for the harness-neutral principles list, see [skill-design-principles.md](skill-design-principles.md). Evidence-breadth corrections (which principles survived, which were amended) are in [research-generalization-audit.md](research-generalization-audit.md).

---

## 1. Executive summary

The system is not a bag of prompts. It is a **small, composable process system** in which an expert engineer's judgment is encoded as artifacts, gates, and decision rights rather than as prose advice. Five findings dominate:

1. **One architectural axis: invocation.** Every skill is either *user-invoked* (only a human can fire it; zero context cost) or *model-invoked* (agent-reachable; its description is a permanently loaded context pointer). This single flag drives composition, documentation strategy, and cost accounting. Evidence: `.agents/invocation.md`, `writing-for-agents/SKILL-MECHANICS.md`. **Confidence: H.**
2. **Two tiers that compose by calling, not by sharing files.** User-invoked workflow skills orchestrate model-invoked discipline skills via explicit Skill-tool instructions; shared reference lives in one owning skill and is reached by invocation, never by cross-folder links. `.agents/invocation.md` L16. **H.**
3. **State lives in artifacts, not conversation.** Cross-session continuity is carried by issue-tracker issues, triage labels, `CONTEXT.md`, ADRs, `.scratch/` tickets, and throwaway branches. Conversation memory is consumed within a session and persisted at its boundary. **H** (with one deliberate exception: `handoff` writes to the OS temp dir by design).
4. **Gates are prose, not machinery.** Planning-before-implementation, reproduction-before-hypothesis, and seams-before-tests are enforced by instructions, phase ordering, and session segmentation — which a model *can* ignore. Only `disable-model-invocation` is harness-enforced. The system makes wrong moves *expensive*, not impossible. **H** (this corrects an initial overclaim).
5. **The system theorizes itself.** `writing-for-agents` is a meta-skill naming its own levers: context load vs cognitive load, the information hierarchy, progressive disclosure, completion criteria, leading words, negation as failure mode. Every other skill was written against it (docs page: "every skill here was written against it"). This makes author *intent* directly citable instead of inferred. **H.**

Known weaknesses are disclosed by the author himself in docs pages (self-reported bugs, stale counts, CONTEXT.md bloat), which both raises trust in the stated philosophy and shows the system's limits are real, not hypothetical. See §8.

---

## 2. System-level mental model

Picture the system as three stacked layers plus a state plane:

```
┌─────────────────────────────────────────────────────────────┐
│ LAYER 3 — ROUTING (human-facing)                            │
│   ask-matt: a router skill that names the others and when   │
│   to reach for each. "It can only hint, never fire them."   │
│   (SKILL-MECHANICS.md)                                      │
├─────────────────────────────────────────────────────────────┤
│ LAYER 2 — WORKFLOW SKILLS (user-invoked)                    │
│   setup → grill-with-docs → to-spec → to-tickets →          │
│   implement → code-review; plus triage, wayfinder, handoff. │
│   Each owns one phase; orchestration is 7–128 line files.   │
├─────────────────────────────────────────────────────────────┤
│ LAYER 1 — DISCIPLINE SKILLS (model-invoked, reusable)       │
│   grilling, domain-modeling, codebase-design, tdd,          │
│   research, prototype, writing-for-agents. "Two             │
│   model-invoked references that run *beneath* the other     │
│   skills, each the single source of truth for its           │
│   vocabulary." (ask-matt/SKILL.md)                          │
├─────────────────────────────────────────────────────────────┤
│ STATE PLANE (durable, outside conversation)                 │
│   issue tracker · triage labels · CONTEXT.md · docs/adr/ ·  │
│   .scratch/ tickets · agent briefs · throwaway branches     │
└─────────────────────────────────────────────────────────────┘
```

Key properties of this model:

- **Skills are phases, not features.** A workflow skill owns exactly one transformation (conversation → spec; spec → tickets; ticket → committed code). ask-matt's canonical chain is `[stated]`: "grill-with-docs sharpens the idea… to-spec (turn the thread into a spec), then to-tickets… kick off implement per ticket, /clearing context between each one… implement builds each issue by driving tdd internally… then closes out by running code-review" (`ask-matt/SKILL.md` L15–28).
- **Context is a managed resource, not a given.** Phase boundaries are the only place context decisions happen: "Mid-phase there is no decision to make" (`ask-matt/PHASE-BOUNDARIES.md`). Planning stays in one unbroken window; each implementation ticket gets a fresh one (`ask-matt/SKILL.md` L32–34). The "~150k-token smart zone" is named explicitly.
- **The human is a scheduled resource.** Decisions queue to the human (grilling rounds, triage "Wait for direction", tdd seam confirmation); facts never do ("Finding facts is your job, never the user's", `grilling/SKILL.md` L20).
- **The repo dogfoods itself** `[observed]`: it has its own `CONTEXT.md`, `.scratch/` issues, `docs/agents/` config, ADRs, and `.out-of-scope/` records — exactly what `setup-matt-pocock-skills` emits into target repos. **Confidence: H.**

---

## 3. Core mindset and principles

Beliefs the authors hold, each with its strongest evidence. `[stated]` quotes are verbatim.

### 3.1 On software engineering
- **Process beats cleverness.** `[stated]` "you need extremely strict and well-defined processes to get them to do useful work" (aihero.dev/5-agent-skills-i-use-every-day). Skills encode "the exact moves I'd make for a given problem so the agent runs them the same way I would, every time" (skills-catalog page). **H.**
- **Standards survive delegation.** `[stated]` "A practical skill system for engineers who want to use AI without giving up their standards" / "They're how I keep my taste and standards intact while the agent does the work." **H.**
- **The codebase constrains agents more than prompts do.** `[stated]` "Your codebase, way more than your prompt or your AGENTS.md file, is the biggest influence on AI's output" (how-to-make-codebases-ai-agents-love). Grey-box modules: "You own the interface. AI owns the implementation. Tests keep it honest." **H.**

### 3.2 On human-agent collaboration
- **Complementarity, not replacement.** `[stated]` "This skill is built to aid an engineer, not to replace one: what comes out tracks the quality of your answers, not the number of questions asked" (skills-grill-me). **H.**
- **Division of labor by epistemic type**: facts to the agent, decisions to the human (`grilling/SKILL.md` L20; wayfinder HITL tickets: "the agent never stands in for the human's side of it"). **H.**
- **Passivity is the collaboration failure mode.** `[stated]` "The failure mode is passivity: answering 'agreed, agreed, agreed' for forty questions and coming out with a plan the agent wrote and you nodded at… the result carries a certainty it hasn't earned" (skills-grill-me). **H.**

### 3.3 On planning before implementation
- **Alignment precedes decomposition precedes code.** `[stated]` Reach for grilling "long before you have worked out what it involves" (skills-grill-me); wayfinder "plans, it does not do: a cleared map hands off, it never carries on into code" (`ask-matt/SCENARIOS.md` L44). **H.**
- **Planning artifacts bind implementation.** Seams agreed in to-spec become tdd's test boundaries ("pre-agreed seams") and code-review's Spec axis — the same artifact is checked at three points. `[observed]` **H.**

### 3.4 On uncertainty and decision-making
- **Fog is mapped, then resolved one decision at a time.** Wayfinder's vocabulary: "fog of war", decision tickets, "never resolve more than one ticket per session" (`wayfinder/SKILL.md` L105). **H.**
- **Uncertain questions get routed by type**: live exchange (HITL grilling), evidence (research sub-agent), or experiment (prototype branch) — `wayfinder/SKILL.md` L77–79. **H.**
- **Ambiguity protocol**: ask if reachable; otherwise default to the branch matching surrounding code *and state the assumption* (`prototype/SKILL.md` L12). **H.**

### 3.5 On testing and verification
- **Verification is a precondition, not a phase at the end.** `[stated]` "It will not let the agent form a theory until a tight feedback loop exists" (skills-diagnosing-bugs); "No red-capable command, no Phase 2." **H.**
- **Tests at interfaces, agreed in advance.** "No test is written at an unconfirmed seam" (`tdd/SKILL.md` L22). **H.**

### 3.6 On code quality and maintainability
- **Deep modules as the design unit.** codebase-design supplies the vocabulary ("deep module", seams); improve-codebase-architecture scans for "deepening opportunities"; tdd borrows it ("call the Skill tool with 'codebase-design' for the vocabulary"). **H.**
- **Vibe coding is the named anti-goal.** `[stated]` "Plug-and-play, not vibe coding" (plugin.json description); "This is still a million miles from vibe coding. You need to apply taste at the boundaries." **H.**

### 3.7 On documentation and knowledge transfer
- **Documents are caches of environment truth.** `[stated]` "The environment is a source of truth too… a document that restates it is a cache… Cache what the agent cannot find by looking: the unwritten convention, the reason behind a choice" (`writing-for-agents/SKILL.md` L79). **H.**
- **Durable artifacts are path-averse.** `[stated]` Agent briefs: "Don't reference file paths: they go stale" (`triage/AGENT-BRIEF.md`); to-spec: "Do NOT include specific file paths or code snippets." **H.**
- **Rejection is also knowledge.** `.out-of-scope/` stores rejected requests for "institutional memory" and "deduplication" (`triage/OUT-OF-SCOPE.md`). **H.**

### 3.8 On controlling agent autonomy
- **Autonomy is granted per skill, per axis, by default-deny.** User-invocation is the deny-by-default posture for anything involving judgment or irreversible action; model-invocation is reserved for disciplines and fact-gathering. `[stated]` selection test: "could the model usefully reach for this autonomously?" (`.agents/invocation.md` L6). **H.**
- **AFK operation is engineered for, not assumed.** research runs as a background agent; agent briefs are written for "a stranger" (AFK agent); wayfinder separates HITL from AFK tickets. **H.**
- **Prose gates are honest about being prose.** diagnosing-bugs opens with "Skip phases only when explicitly justified" — the system licenses deviation under justification rather than pretending gates are mechanical. **H.** `[recommended]` Keep this honesty: do not describe prose gates as enforcement.

---

## 4. Recurring skill-design patterns

Patterns observed across the corpus of 35 skills (25 promoted). Confidence per pattern; counterexamples included.

### P1. The frontmatter decision is the design decision `[stated + observed]` — **H**
`name`, `description`, optional `disable-model-invocation`. Nothing else (argument-hint appears on 3 of 35 skills, all conversation-compacting ones: handoff, teach, claude-handoff). Invocation choice is a budget trade: model-invoked = "permanent context load in exchange for discoverability"; user-invoked = "zero context load, but it spends cognitive load: you are the index" (`SKILL-MECHANICS.md` L9–10).

### P2. Dual description registers `[stated]` — **H**
Model-invoked descriptions carry trigger branches in user phrasing: "Use when the user says 'diagnose'/'debug this', or reports something broken/throwing/failing/slow." User-invoked descriptions are stripped human summaries: "Stop. That last message did not land: re-pitch it." Negative triggers exist but are rare (exactly one in corpus: wizard's "Don't invoke this for steps the agent can perform itself").

### P3. Progressive disclosure via lazy links `[stated + observed]` — **H**
SKILL.md holds decision logic; formats/examples/scaffolds live in linked files loaded when a branch fires: "See [tests.md](tests.md) for examples" (tdd); branch-conditional loading ("'Does this logic feel right?' → [LOGIC.md](LOGIC.md)", prototype). **Correction to the naive reading** `[stated]`: the author disavows pure token motives — "Not primarily a token optimisation: it is how the hierarchy is protected" (`writing-for-agents/SKILL.md` L39). The operating rule is the **branching test**: "inline what every branch needs, and push behind a pointer what only some branches reach." Counterexamples that follow the rule: to-spec inlines its 55-line spec template (every invocation needs it); code-review inlines the smell baseline because sub-agents "have no other access to it".

### P4. Composition by Skill-tool invocation, never file links `[stated]` — **H**
"Dependencies are expressed as an explicit instruction to call the Skill tool with the named skill…, not deep ../other-skill/FILE.md cross-references" (`.agents/invocation.md` L16). Router shells can be one line: grill-me's entire body is "Call the Skill tool with 'grilling'." Convention nuance `[observed]`: the flagship `implement` skill violates the wording rule ("Use /tdd where possible" — bare slash prose), which the repo's own changeset machinery later standardized against. **Confidence: H.**

### P5. Numbered phases with explicit gates `[observed]` — **H**
diagnosing-bugs: six phases, checkbox completion lists, "Do not proceed until you have reproduced **and** minimised." triage: labeled state machine with "flag transitions that look unusual and ask before proceeding." wayfinder: throughput gate ("never resolve more than one ticket per session"). Gate density tracks **irreversibility of a wrong run** `[interpretation]`, not skill length: tdd is 39 lines but carries one of the hardest gates; teach is 140 lines and gateless (low-stakes). Counterexamples preserved, not smoothed over. **M.**

### P6. Question protocols with format, batching, and recommended answers `[stated]` — **H**
grilling: numbered questions, whole frontier per round, mandatory recommended answer, fixed ❓/➡️ visual format, wait-for-answer semantics. setup: "One section, one answer… Lead each section with the recommended answer so the user can accept it in a word." triage: "Questions must be specific and actionable, not 'please provide more info'." tdd prescribes the exact question verbatim. Deliberate opt-out: to-spec **forbids** interviewing ("Do NOT interview the user; just synthesize what you already know") — proof the protocol is a tool, not a default.

### P7. Sub-agent contract `[observed]` — **H**
- Parallel dispatch to protect contexts: "Both axes run as parallel sub-agents so they don't pollute each other's context" (code-review).
- Self-contained briefs: shared material "pasted in full (the sub-agent has no other access to it)".
- Sub-agents return evidence; the parent synthesizes: "Present the two reports… verbatim or lightly cleaned. Do not merge or rerank findings."
- Pre-flight validation before dispatch: "A bad ref or empty diff should fail here, not inside two parallel sub-agents."
- Fact-only delegation from interviews: grilling dispatches sub-agents for facts, never for decisions, and does not block on them.

### P8. Grounding in repo context before asking `[stated + observed]` — **H**
Boilerplate "(if it exists)" reads of CONTEXT.md (tdd, diagnosing-bugs); concrete probe lists ("Read the repo first, don't ask cold", wizard); git history as lookup (`git log --oneline` for hot spots); config-as-truth with the shared redirect sentence "If not, tell the user to run `/setup-matt-pocock-skills`" (5 skills; 4 share the form verbatim, code-review is a conditional variant).

### P9. Stopping criteria as checkable-and-exhaustive bounds `[stated]` — **M** (doctrine stated; practice mixed)
Doctrine: "Every step ends on a completion criterion… The strongest criteria are both checkable and exhaustive" (`writing-for-agents` L47–52). Practice: strong in wizard/to-questionnaire/diagnosing-bugs ("Done when:" clauses, `- [ ]` checklists, "the frontier is empty"), absent in implement/to-spec/handoff/teach. Treat as a strong norm, not a universal structure.

### P10. Failure handling menu `[observed]` — **H**
Missing config → redirect to setup. Missing doc → create lazily ("Create files lazily: only when you have something to write", domain-modeling). Cannot proceed → structured escalation ("Stop and say so explicitly. List what you tried. Ask the user for (a)… (b)… (c)…", diagnosing-bugs). Optional axis missing → degrade with a report (code-review Spec axis: "no spec available"). Input didn't land → scripted re-pitch (wait-what). Skill unnecessary → abort gracefully (wayfinder: "If this surfaces no fog… you don't need a map. Stop.").

### P11. Vocabulary engineering `[stated]` — **H**
Leading words recruit pretrained priors: "fast, deterministic, low-overhead → tight"; "a loop you believe in → red, turning a fuzzy gate into a binary observable state" (`writing-for-agents` L63–72). The corpus is dense with them: tight, red, seam, frontier, fog of war, tracer bullet, deep module. Coined terms are bolded on first use and reused plain thereafter `[observed]`.

### P12. Persona and voice `[observed]` — **M**
Imperative second person throughout; persona set by one strong word ("relentless" — the author explicitly says a weaker word "is a no-op, and the fix is a stronger word"); essence-first definitions ("A prototype is throwaway code that answers a question"); role limits ("Your job is only to scope the procedure and author its stages", wizard).

### P13. Durability-aware artifact design `[observed]` — **H**
Durable artifacts avoid volatile references (no file paths/line numbers in agent briefs); ephemeral artifacts are placed to avoid polluting repos (handoff and HTML reports to OS temp dir; prototypes to throwaway branches with a context pointer back). Placement is a deliberate durability statement, not an accident.

---

## 5. End-to-end workflow architecture

### 5.1 The main flow `[stated]`

```
setup-matt-pocock-skills        (run once; emits docs/agents/* config)
        │
        ▼
grill-with-docs                 (interview → CONTEXT.md terms + ADRs)
        │
        ├── multi-session? ──► wayfinder (decision-ticket map; plans, doesn't do)
        │                          └── merges back at to-spec
        ▼
to-spec                         (spec issue + ready-for-agent label)
        ▼
to-tickets                      (tracer-bullet tickets w/ blocking edges;
                                 human approves breakdown)
        ▼
implement  ×N                   (one fresh session per ticket)
   ├── drives tdd               (red-green slices at pre-agreed seams)
   └── ends with code-review    (Standards ∥ Spec parallel sub-agents)
        ▼
commit
```

### 5.2 On-ramps and side-channels `[stated]`
- **Incoming work**: triage moves external issues to `ready-for-agent` with an agent brief "the contract" — implement picks them up (`ask-matt/SKILL.md` L40).
- **Big unknowns**: wayfinder charts decision tickets; resolution types: HITL grilling, research sub-agent, prototype branch (`wayfinder/SKILL.md` L75–79).
- **Broken code**: diagnosing-bugs; if "no correct seam exists, that itself is the finding" → hands off to improve-codebase-architecture (`diagnosing-bugs/SKILL.md` L120, `ask-matt` L44).
- **Maintenance**: improve-codebase-architecture → picking a finding "generates an idea you can take into the main flow at grill-with-docs".
- **Unreachable experts**: to-questionnaire → "What comes back is material for grill-with-docs or to-spec."

### 5.3 Handoff mechanics: artifacts, not memory `[observed]` — **H**
| Boundary | Artifact | Location |
|---|---|---|
| interview → memory | glossary terms, ADRs | `CONTEXT.md`, `docs/adr/` (lazy creation) |
| plan → build | spec + `ready-for-agent` label | issue tracker |
| spec → sessions | tickets with blocking edges | tracker or `.scratch/<slug>/issues/` |
| session → session | handoff doc (deliberately ephemeral) | OS temp dir |
| review → record | two-axis report citing standards/spec lines | conversation + commit |
| rejection → memory | out-of-scope record | `.out-of-scope/` |
| experiment → later | prototype branch + context pointer | `prototype/<name>` branch |

`ready-for-agent` is the system's shared handoff token, produced by three skills (to-spec, to-tickets, triage) `[observed]`.

**Nuance found by the critic**: conversation memory *is* consumed within a session (to-spec "just synthesize what you already know"); artifacts mediate only the *cross-session* boundary. And handoff's temp-dir placement shows ephemerality can be the correct durability choice. **H.**

### 5.4 Entry/exit discipline `[observed]` — **H**
- Entry preconditions: five skills gate on setup config with a redirect; tdd gates on confirmed seams; diagnosing-bugs gates on a red-capable loop.
- Exit criteria: grilling ("frontier is empty"), wayfinder ("the way is clear"), diagnosing-bugs (checkbox list incl. "instrumentation removed"), SCENARIOS cards each carry an "Exit condition" field.
- Counterexamples (no explicit exit in own SKILL.md): implement, code-review, to-spec, to-tickets, handoff, research, prototype, teach — some only have exits in SCENARIOS.md cards.

### 5.5 Context-window management `[stated]` — **H**
"Keep steps 1–3 in one unbroken context window (don't compact or clear until after to-tickets)"; "Each implement then starts fresh, working from the ticket"; at the smart-zone limit "compact at the nearest phase boundary" (`ask-matt/SKILL.md` L32–34). The phase-boundary decision tree orders five options — Continue → /clear → /handoff → Subagent → /compact — "the first yes wins" (`PHASE-BOUNDARIES.md`).

---

## 6. Human versus agent responsibility model

`[stated + observed]` — **H**. The system draws the line by *epistemic type*, not by task size:

| Responsibility | Owner | Enforcement mechanism | Evidence |
|---|---|---|---|
| Facts (lookup, scan, repro, read) | Agent | "don't ask the user for anything you could look up yourself"; sub-agent dispatch | grilling L20; wizard "Read the repo first, don't ask cold" |
| Decisions (scope, seams, categories, direction) | Human | Wait-gates; recommended answers lower the cost of deciding | grilling L20–22; triage "Wait for direction"; tdd "No test is written at an unconfirmed seam" |
| Judgment calls under ambiguity | Human if reachable; else agent defaults + discloses | Stated assumption requirement | prototype L12; diagnosing-bugs AFK fallback |
| Taste/standards | Human, encoded in advance | Standards docs consumed by code-review's Standards axis; CONTEXT.md vocabulary | code-review; writing-for-agents |
| Irreversible actions | Human or mechanical guard | user-invocation flag; git hook script that tells the agent "it does not have authority" | invocation.md; git-guardrails scripts/ |
| Only-humans-can steps (credentials, dashboards) | Human, via generated script | wizard generates a bash wizard the agent "never runs end-to-end" | wizard SKILL.md |

Two refinements worth adopting:
1. **Recommended answers change the economics of waiting.** Every question ships with a recommendation the user can accept in a word (setup: "Lead each section with the recommended answer"). Checkpoints stay cheap; that is why the system can afford many of them. `[observed]`
2. **AFK is a first-class operating mode.** HITL/AFK ticket classification, agent briefs written for strangers, background research agents. The human need not be synchronous. `[stated]`

---

## 7. Quality gates and failure-prevention mechanisms

Mapped to the seven failure classes you asked about:

### 7.1 Premature coding
- Grilling must empty its frontier and get explicit confirmation: "Do not act on it until the user confirms you have reached a shared understanding" (`grilling/SKILL.md` L22). **H.**
- wayfinder "plans, it does not do"; its map hands off at to-spec — "Looping the map straight into /implement skips that collapse" (`ask-matt/SKILL.md` L48). **H.**
- diagnosing-bugs: "jumping straight to a hypothesis is the exact failure this skill prevents." **H.**

### 7.2 Vague requirements
- Interview format forces precision: one idea per question, answer stubs, most-important-first (to-questionnaire); triage: questions "specific and actionable". **H.**
- to-spec refuses to carry volatile detail ("Do NOT include specific file paths or code snippets. They may end up being outdated very quickly") — vagueness-by-rot is also guarded against. **H.**
- Glossary challenge pattern: "Your glossary defines 'cancellation' as X, but you seem to mean Y. Which is it?" (domain-modeling). **H.**

### 7.3 Plausible but incorrect output
- Evidence bars: code-review findings must "cite the standard (file + the rule)" and "Quote the spec line for each finding"; research writes "citing each claim's source… Follow every claim back to the source"; teach: "Never trust your parametric knowledge", "Lessons should be littered with citations". **H.**
- Reproduction before theory; minimisation before fix ("Do not proceed until you have reproduced **and** minimised"). **H.**
- Verification commands as exit gates: `bash -n`, shellcheck, project typecheck/tests/format (wizard, resolving-merge-conflicts). **H.**

### 7.4 Uncontrolled scope expansion
- Seam confinement: "Test only at pre-agreed seams" (tdd); resolving-merge-conflicts: "Do not invent new behaviour. Always resolve; never --abort." **H.**
- Throughput caps: wayfinder's one-ticket-per-session; wizard's "never invent steps that may not exist". **H.**
- Scope-creep knowledge base: `.out-of-scope/` with a dedup-poisoning guard (don't record wontfix-because-already-implemented). **H.**

### 7.5 Shallow code review
- Two axes run in parallel sub-agents "so they don't pollute each other's context"; aggregation forbidden from reranking: "Don't pick a single winner across axes: that's the reranking the separation exists to prevent." **H.**
- Pre-flight validation prevents reviewing nothing; the 12-item smell baseline is pasted in full into sub-agent briefs. **H.**

### 7.6 Implementation diverging from specification
- The spec is a live checkpoint, not an input: code-review's Spec axis re-quotes the spec against the diff; tdd's seams are the spec's seams; agent briefs are "the contract" for AFK agents. `[observed]` **H.**
- Known gap `[observed]`: implement never closes or verifies tickets (docs say this is "expected" behavior); a divergence risk the author accepts. Report as a tension, §8.

### 7.7 Loss of context between agents or sessions
- Artifact persistence (§5.3), phase-boundary context rules (§5.5), handoff's "suggested skills" section propagating invocation advice forward (`handoff/SKILL.md` L10), CONTEXT.md read at skill start, "Don't re-ask resolved questions" (triage L112). **H.**
- Sub-agent context isolation is also a loss-prevention measure in reverse: parallel agents "don't pollute each other's context". **H.**

---

## 8. Tensions, trade-offs, and exceptions

Preserved deliberately; these are real and instructive, not noise to smooth away.

1. **Negation doctrine vs negation practice.** `[stated]` writing-for-agents calls negation a failure mode: "steering by prohibition drags the forbidden behaviour into context… the ban half-reads as an instruction to do the thing." `[observed]` Yet the corpus is saturated with prohibitions ("never --abort", "No test is written at an unconfirmed seam", wizard's description is itself a negative trigger). Best reading: the doctrine's escape hatch ("a prohibition earns its place only as a hard guardrail") is being applied wholesale — the corpus treats nearly everything as a hard guardrail, which blunts the doctrine. `[interpretation]` **H.**
2. **Prose gates pretending to be structure.** Only `disable-model-invocation` is harness-enforced; every other gate is prose a model can ignore — and diagnosing-bugs openly licenses skipping ("Skip phases only when explicitly justified"). Trade-off accepted: flexibility + justification requirement over mechanical certainty. **H.**
3. **Waiting doctrine vs AFK engineering.** The system preaches "put each decision to them and wait", then engineers around it: AFK hypothesis ranking, non-blocking sub-agent fact-finding, background research. The actual rule is narrower than the rhetoric: *decisions* wait; *work* doesn't. **H.**
4. **Convention drift in cross-references.** `.agents/invocation.md` codifies "call the Skill tool with X" wording; `implement` (the flagship flow skill) ships bare "/tdd" prose; a changeset later standardized ten skills. Rules in this repo sometimes post-date or ignore their most prominent violators. **H.**
5. **Docs lag the repo.** The web says "thirteen of the plugin's twenty-two skills carry the flag"; the repo has 25 plugin skills, 14 user-invoked. Six advertised category pages 404. The index links dead names (/to-prd, /to-issues, /domain-model). Three different install stories coexist on the site. Per your conflict rule, the repo wins; the docs are a stale cache — which is exactly the failure mode `writing-for-agents` warns about. **H.**
6. **Self-disclosed operational bugs.** Docs pages admit: codebase-design once burned 100k tokens redesigning unrequested code (no stopping rule, issue #449); research nesting cost ~450k tokens (#530); to-tickets' native GitHub blocking links "not actually created" (#554, #513); teach can write into the skill install dir (#377); CONTEXT.md bloat is "the most-reported problem" and guidance is "not yet strong enough to prevent the growth". Lesson: even disciplined systems need stopping rules on model-invoked references and mechanical verification of claimed side effects. **H.**
7. **Invocation-type churn.** wizard graduated and was *made* model-invoked; writing-great-skills flipped to model-invoked on rename. The axis is stable as architecture, but individual assignments are actively traded. Treat invocation type as a maintained decision, not a fixed property. **H.**
8. **The glossary bent around a skill.** CONTEXT.md says avoid "ticket", then carves out "Decision ticket" after wayfinder forced the word in. Vocabulary governance is real but yields to shipping pressure. **H.**
9. **Durability is contextual, not maximal.** handoff writes to the OS temp dir on purpose; agent briefs forbid file paths. The system optimizes for *appropriate* lifespan, not permanence. **H.**
10. **Frequency is not proof.** "Done when:" appears in roughly half the skills; teaching is gateless; misc breaks the README grouping convention (sanctioned by CLAUDE.md). Patterns above are labeled accordingly; where a pattern is frequency-only (no mechanism), it says so. `[recommended]` Apply the same standard to your analyses. **H.**

---

## 9. Skill Design Constitution (12 principles)

Harness-neutral. Each principle: statement, why (evidence), and the failure it prevents. `[recommended]` — derived, not quoted.

1. **Decide who can fire it before writing a word of it.** Invocation reach (human-only vs agent-reachable) is the first design decision; it determines description style, composition possibilities, and context cost. *Why*: the entire system pivots on this one axis (`invocation.md`; `SKILL-MECHANICS.md`). *Prevents*: accidental autonomy on judgment-heavy workflows; silent context tax.
2. **Budget two loads explicitly: context load and cognitive load.** Every line that stays loaded costs attention every turn; every skill only a human remembers costs human memory. Trade deliberately; when human-index load piles up, cure it with a router skill. *Why*: "the two loads" doctrine (`writing-for-agents` L20–27); ask-matt exists for exactly this. *Prevents*: bloat and forgotten tooling.
3. **Encode expert judgment as gates, artifacts, and decision rights — not as advice.** Write the move, the stopping condition, and who decides; skip motivational prose. *Why*: "the exact moves I'd make… every time" (skills-catalog); every quality gate in §7 is a move, not a hope. *Prevents*: plausible-but-wrong output; drift between runs.
4. **Facts are the agent's job; decisions are the human's; never blur them.** Route lookups to tools/sub-agents; route choices to the human with a recommended answer attached so deciding costs one word. *Why*: grilling L20; recommended-answer convention across grilling/setup/to-questionnaire. *Prevents*: passivity spirals and user fatigue.
5. **State crosses session boundaries only as artifacts.** Pick each artifact's home and lifespan deliberately: durable contract (tracker, brief), living memory (glossary, ADRs), or deliberately ephemeral (temp-dir handoff). Avoid volatile references (paths, line numbers) in anything durable. *Why*: §5.3; AGENT-BRIEF's path ban. *Prevents*: context loss; rotting handoffs.
6. **Make every step end on a completion criterion that is checkable and exhaustive.** Prefer observable states ("the loop goes red") over fuzzy ones ("understanding reached"); if the bound is irreducibly fuzzy, name the rush you're defending against. *Why*: `writing-for-agents` L47–52; diagnosing-bugs checkboxes. *Prevents*: premature completion.
7. **Protect the information hierarchy: steps on top, reference behind pointers.** Inline what every branch needs; disclose what only some branches reach. Branching is the disclosure test. Do not justify this as token-saving alone — it is what keeps the steps legible and attended to. *Why*: `writing-for-agents` L31–43; tdd/prototype link patterns. *Prevents*: sprawl; attention coin-flips on the steps.
8. **Compress with leading words, not sentences.** Prefer a compact pretrained term defined once (tight, red, seam) over repeated restatements; coining is allowed only with an explicit definition. *Why*: `writing-for-agents` L63–72. *Prevents*: token waste; unstable behavior across runs.
9. **Prompt the positive; reserve prohibitions for hard guardrails, paired with the target behavior.** *Why*: negation-as-failure-mode doctrine (L74) — and tension §8.1 warns against overusing the escape hatch. *Prevents*: forbidden behavior becoming more available.
10. **Degrade by protocol, never by improvisation.** For every external dependency, write the missing-case branch: redirect to setup, create lazily, skip-and-report, or escalate with a structured menu of what you need. *Why*: pattern P10 evidence across five skills. *Prevents*: invented config; silent divergence.
11. **Sub-agents get self-contained briefs and return evidence, never conclusions.** Validate inputs before dispatch; keep parallel agents context-isolated; aggregate without reranking. *Why*: code-review's full contract. *Prevents*: polluted contexts; laundered synthesis.
12. **Keep the environment as source of truth; treat documents as caches with expiry risk.** Restate only what cannot be looked up (the convention, the reason, the gotcha); prune lines that stopped bearing on the task; hunt no-ops sentence by sentence. *Why*: `writing-for-agents` L78–81; the repo's own stale docs (tension §8.5) demonstrate the failure. *Prevents*: sediment; authoritative-looking staleness.

---

## 10. Practical checklist for creating a new skill

**A. Before writing**
- [ ] Named the failure mode this skill prevents (if none, no skill needed).
- [ ] Decided invocation type using the test: *could the model usefully reach for this autonomously, or must another skill call it?* No → human-only.
- [ ] Identified the leading word(s) the skill runs on; they exist in your own prompts.
- [ ] Mapped inputs consumed and artifacts produced; each artifact has a home and a lifespan.
- [ ] Listed external dependencies and each one's missing-case branch.

**B. Structure**
- [ ] Frontmatter: name = directory; description written in the right register (trigger branches if model-reachable, human summary if not).
- [ ] Steps in order, each ending on a checkable completion criterion ("Done when:").
- [ ] Gates placed where a wrong run becomes expensive or irreversible; gates phrased honestly (defaults vs absolutes).
- [ ] Decision points routed to the human, each with a recommended answer.
- [ ] Facts routed to tools/sub-agents; no question asks the user for a lookup.
- [ ] Bulky reference disclosed behind links with a "what you'll find" gloss; only what every branch needs is inline.
- [ ] Cross-skill needs expressed as "call the Skill tool with 'X'" — only if X is model-reachable.

**C. Guardrails**
- [ ] Entry precondition stated; missing-dependency behavior defined (redirect / lazy-create / degrade / escalate).
- [ ] Stopping criterion for the whole skill (exhaustive: what must all be true?).
- [ ] Prohibitions limited to hard guardrails, each paired with the positive target.
- [ ] Durable artifacts contain no file paths, line numbers, or volatile snippets.

**D. After writing**
- [ ] Pruned: every sentence changes behavior versus the model's default (no-op test).
- [ ] Ran the skill end-to-end once; checked the criterion against the actual run, not the intended run.
- [ ] If the skill joins a workflow: the router/map that names it is updated; the handoff token it emits is named.
- [ ] If it replaces something: the replacement is recorded where the old thing is removed.

---

## 11. Generic skill template (derived)

```markdown
---
name: <kebab-case-name>
description: <one line. Model-reachable: "What it does. Use when <trigger
  branch 1>, <trigger branch 2>." Human-only: plain summary, triggers
  stripped. Front-load the leading word.>
disable-model-invocation: true   # only if human-only
---

<One or two sentences: the essence. Define the leading word(s).
 "A <skill> is <defining property>. The <input> decides the shape.">

<Precondition block (if any):
 Requires <config/artifact>. If missing, <redirect to setup / create
 lazily / stop and say so — pick the protocol, don't improvise.>>

## Steps

1. **<Step name>**
   <Imperative instructions. Ground in repo context before asking:
   read <file/config> (if it exists); look up what you can; never ask
   the user for a fact.>
   <If a decision: present options + your recommended answer. Wait.>
   **Done when:** <checkable, exhaustive criterion.>

2. **<Step name>**
   ...
   (Gate: "Do not proceed to <next> until <observable state>."
    If deviation is ever legitimate, say so: "Skip only when
    explicitly justified.")

<Sub-agent steps, if any:
 Dispatch with a self-contained brief (<paste shared material in full;
 the sub-agent has no other access to it>). Sub-agents return findings;
 you synthesize. Validate inputs before dispatch.>

## Failure branches

- <Missing X> → <redirect | create lazily | skip-and-report | escalate:
  "Stop. List what you tried. Ask for (a)… (b)…">
- <Ambiguous input> → ask if reachable; else default and state the
  assumption.

## Artifacts

- Produces: <artifact> → <location> (<durable | living | ephemeral>;
  no file paths or line numbers in durable outputs)
- Consumes: <inputs and where they come from>

## Completion

The skill is done when <exhaustive terminal state>, and <human
confirmation if the output carries decisions>.
```

Supporting-file rule: when a format/template/example is large or branch-specific, move it to a sibling `*.md` and link it with a one-line gloss ("Use the format in [X-FORMAT.md](X-FORMAT.md)"). Shared reference used by several skills lives in ONE skill that others call — never copied, never cross-linked as files.

---

## 12. Scoring rubric for your future skills

Moved to its own file: [skill-scoring-rubric.md](skill-scoring-rubric.md) — 12 axes scored 0–3, pass bar ≥ 24/36 with no axis at 0, plus an axis glossary tracing each axis to its evidence in §13. Quick diagnostics: axis 2 (Failure mode) at 0 means it's a document, not a skill; axis 4 (Decision routing) at 0 reproduces the passivity failure mode; axis 11 (Honesty) is the one this repo models best and most skill collections ignore.

---

## 13. Evidence matrix

Conclusion → primary sources → kind of evidence → confidence.
R = repository (implementation evidence, wins conflicts); W = web documentation; I = interpretation.

| # | Conclusion | Sources | Kind | Conf |
|---|---|---|---|---|
| 1 | Invocation type is the primary architectural axis | R: `.agents/invocation.md` L3, `SKILL-MECHANICS.md` L7–12, bucket READMEs, frontmatter of all 35 skills; W: skills-implement page | R stated + verified | H |
| 2 | Two-tier composition via Skill-tool calls; user-invoked never calls user-invoked | R: `invocation.md` L8/L16/L22; grep of all "call the Skill tool" targets (all model-invoked); grill-me/grill-with-docs bodies | R stated + verified; one wording violation (implement) | H |
| 3 | Cross-session state is artifact-mediated; conversation is in-session only | R: wayfinder L21/L125, AGENT-BRIEF.md L3, to-tickets L62–63, handoff L8, domain-modeling L40/L62, `.out-of-scope/` | R stated; nuance from critic | H |
| 4 | Gates are prose + session design, not machinery (planning-before-implementation is procedural) | R: diagnosing-bugs L8 ("Skip phases only when explicitly justified"), tdd L22, wayfinder Notes override, PHASE-BOUNDARIES.md | I (corrected overclaim; harness only enforces invocation flag) | H |
| 5 | Fact/decision split with wait-gates and recommended answers | R: grilling L20–22; triage L72; setup "Lead each section with the recommended answer"; W: skills-grilling ("Decisions are yours, and it must wait for them") | R+W stated | H |
| 6 | Description = always-loaded context pointer; dual registers | R: `SKILL-MECHANICS.md` L9–10; description corpus comparison | R stated + verified | H |
| 7 | Progressive disclosure protects hierarchy; branching test governs inlining | R: `writing-for-agents` L31–43; tdd/prototype/domain-modeling links; counterexamples (to-spec template, code-review baseline) follow the rule | R stated | H |
| 8 | Completion criteria doctrine; practice mixed | R: `writing-for-agents` L47–52 (doctrine); wizard/diagnosing-bugs vs implement/teach (practice) | R stated doctrine, observed practice | M |
| 9 | Standardized failure handling (redirect/lazy/degrade/escalate) | R: 5 setup-redirect sites; domain-modeling L40; code-review "no spec available"; diagnosing-bugs escalation menu | R stated | H |
| 10 | Vocabulary engineering via leading words; glossary-only CONTEXT.md | R: `writing-for-agents` L63–72; domain-modeling L64; CONTEXT.md structure; bolded coinings across corpus | R stated + observed | H |
| 11 | Quality gates per failure class (§7) | R: sources listed in §7 per gate | R stated | H |
| 12 | Sub-agent contract (self-contained briefs, evidence-only, isolation, pre-flight) | R: code-review SKILL.md; DESIGN-IT-TWICE.md; grilling L20; PHASE-BOUNDARIES.md | R stated | H |
| 13 | Repo dogfoods its own system | R: root CONTEXT.md, `.scratch/scenario-playbook`, docs/agents/*, `.out-of-scope/`, `.agents/adr/` | R observed | H |
| 14 | Gate density tracks irreversibility of a wrong run | R: tdd (short, hard gate) vs teach (long, gateless) vs diagnosing-bugs | I (critic-corrected; no author statement) | M |
| 15 | "Fleet of middling-to-good engineers with no memory" mental model | W: 5-agent-skills-i-use-every-day (quote); R: session/artifact design consistent | W stated, R consistent | M |
| 16 | Author self-discloses operational limits (token blowups, bloat, unfixed bugs) | W: skill pages' issue references (#449, #530, #554, #377, CONTEXT.md bloat) | W stated | H |
| 17 | Docs lag repo: counts (22 vs 25 plugin skills; 13 vs 14 flagged), dead category pages, dead names, three install stories | R: plugin.json (25 entries, 14 flagged) vs W: ask-matt page, index, catalog | Conflict: R wins | H |
| 18 | The system theorizes itself; intent is citable, not inferred | R: `writing-for-agents/SKILL.md`, `SKILL-MECHANICS.md`, `.agents/invocation.md`, `.agents/writing-docs.md`, `.agents/adr/*`; W: "every skill here was written against it" | R+W stated | H |

### Unverifiable / excluded claims
- Changelog bodies ("63% Token Reduction") — pages 404; treated as index-summary only.
- All GitHub issue numbers cited by web docs — issue tracker not inspected in this session; reported as the docs' own claims.
- The six missing category pages ("The Main Flow", "Reference Skills", etc.) — no public content exists; any workflow narrative rests on the repo, which is complete.

---

## Appendix: Working glossary (this analysis)

Per the grill-with-docs protocol, terms resolved during this session, kept glossary-only:

- **User-invoked skill**: a skill only the human can fire (`disable-model-invocation: true`); zero context load, spends cognitive load.
- **Model-invoked skill**: an agent-reachable skill; its description is a permanently loaded context pointer; other skills may call it.
- **Discipline skill**: a model-invoked skill encoding a reusable practice (grilling, domain-modeling, codebase-design); runs *beneath* workflow skills.
- **Workflow skill**: a user-invoked skill owning one phase of the pipeline; composes discipline skills.
- **Router skill**: a user-invoked skill that names the others and when to reach for each; hints, never fires.
- **Leading word**: a compact pretrained term that anchors a region of behavior (tight, red, seam, frontier).
- **Context pointer**: a reference held in context that names out-of-context material and encodes when to reach it.
- **Context load / cognitive load**: the two budgets — always-loaded tokens vs human memory of what exists.
- **Frontier**: the set of decisions whose prerequisites are settled; the grilling unit of work.
- **Seam**: a pre-agreed interface boundary where tests are written.
- **Handoff token**: a durable marker that work is ready to cross a boundary (`ready-for-agent`).
- **Phase boundary**: the gap between phases; the only place context-management decisions belong.

---

*Method note: four analyst sub-agents (Repository, Workflow, Skill Design, Documentation) gathered evidence in parallel; a Critical Reviewer challenged 15 consolidated conclusions across 8 targeted checks, downgrading 2 overclaims (structural enforcement; token-budgeting motive) and 4 partials; the lead agent verified writing-for-agents, SKILL-MECHANICS.md, CONTEXT.md, and plugin.json directly before synthesizing. No conclusion above rests on a single sub-agent's unverified claim.*
