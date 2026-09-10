# Skill Scoring Rubric

## How to use

Score each axis 0–3 against the skill you are evaluating.

- **Pass bar: ≥ 24/36, with no axis at 0.**
- A 0 on axis 2 (Failure mode) means you wrote a document, not a skill.
- A 0 on axis 4 (Decision routing) reproduces the passivity failure mode the source system exists to prevent.
- Axis 11 (Honesty) is the one the source repo models best (self-disclosed bugs) and most skill collections ignore entirely.

## Rubric

| Axis | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| **1. Invocation fit** | Reach undefined | Type chosen but description register wrong | Correct type + register | Type, register, and reach justified against "can the model usefully fire this alone?" |
| **2. Failure mode** | None named | Vague goal | Named failure mode | Failure mode + the gate(s) that specifically block it |
| **3. Completion criteria** | None | Vague bound ("when done") | Checkable per step | Checkable AND exhaustive; fuzzy bounds defended |
| **4. Decision routing** | Agent decides everything, or asks everything | Split exists but blurry | Facts→agent, decisions→human | Split + recommended answers + wait semantics |
| **5. Artifact discipline** | State in conversation only | Artifacts exist, no home/lifespan | Home + lifespan per artifact | + durability hygiene (no volatile refs; placement deliberate) |
| **6. Context economy** | Everything inline | Some disclosure | Branching test applied | Leading words + pruned no-ops + hierarchy protected |
| **7. Failure branches** | Improvises | One catch-all | Per-dependency protocol | Redirect/lazy/degrade/escalate all covered |
| **8. Composability** | Standalone island | Mentions other skills vaguely | Calls model-reachable skills correctly; emits a named handoff token | Entry/exit + handoff token documented; router updated |
| **9. Voice & words** | Generic ("be thorough") | Imperative but weak words | Strong leading words, defined | Essence-first definition; positive prompts; guardrails paired with targets |
| **10. Testability** | Never run once | Run once | Run against a real case; criteria checked | Run + drift-checked across repeated runs; no-op test applied |
| **11. Honesty** | Claims mechanical enforcement | — | Prose gates labeled as defaults | Known limits/exceptions disclosed in the skill itself |
| **12. Pruning** | >150 lines of steps | Long but organized | Steps lean; reference disclosed | Every line passes the relevance + no-op tests |

## Axis glossary

What the source system's evidence means by each axis's key terms:

1. **Invocation fit** — who can fire the skill (human-only vs agent-reachable) decides description style, composition possibilities, and always-loaded context cost. 
2. **Failure mode** — the skill exists to block a specific failure (premature hypothesizing, passivity, shallow review). Gates are the moves that block it, not hopes.
3. **Completion criteria** — "the strongest criteria are both checkable and exhaustive"; prefer observable states ("the loop goes red") over fuzzy ones ("understanding reached").
4. **Decision routing** — "Finding facts is your job, never the user's… The decisions are the user's: put each to them and wait"; recommended answers make deciding cost one word.
5. **Artifact discipline** — state crosses session boundaries only as artifacts, each with a deliberate home and lifespan; durable artifacts carry no file paths or line numbers (they go stale).
6. **Context economy** — inline what every branch needs, disclose what only some reach (branching test); compress repeated ideas into defined leading words; prune sentences that are no-ops versus the model's default.
7. **Failure branches** — missing dependencies handled by protocol, never improvisation: redirect to setup, create lazily, skip-and-report, or structured escalation.
8. **Composability** — skills call model-reachable skills explicitly ("call the Skill tool with X") and emit named handoff tokens (e.g. `ready-for-agent`); the router that names them stays in sync.
9. **Voice & words** — imperative second person; strong leading words beat weak ones ("relentless" not "thorough"); prohibitions reserved for hard guardrails, paired with the positive target.
10. **Testability** — run the skill against a real case and check the criterion against the actual run; re-run to catch drift across repeated use.
11. **Honesty** — prose gates are labeled as defaults ("Skip phases only when explicitly justified"), not sold as mechanical enforcement; known limits are disclosed.
12. **Pruning** — steps stay lean; bulky or branch-specific reference lives behind links; every line must still bear on what the skill does.
