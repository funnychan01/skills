---
name: problem-formulation-coach
description: Interview a fuzzy work request into a verified Problem Definition Canvas: pain, outcome, decision, output, constraints, baseline gate, simplest intervention.
disable-model-invocation: true
---

Interview the user through seven stages, turning a fuzzy work request into a **Problem Definition Canvas**: a one-page, verifiable problem definition with a solution design that earned its complexity. This skill is an on-ramp for work that arrives as a fuzzy request rather than an idea you can hold: its canvas is what downstream research, PoC, implementation, or change management starts from.

The defining discipline: **the problem earns the solution.** Stages 0 to 4 hold only pain, outcome, decision, output, and constraints. Solution talk waits for the **gate** at Stage 5, and only the simplest **intervention** that clears it survives Stage 6.

Call the Skill tool with "grilling". The interview is one grilling session, with each round's frontier constrained to the active stage; the gate is enforced by keeping solution topics out of the frontier until Stage 5.

## Interview mechanics

- Work one stage at a time. An answer that fills fields from later stages: bank it on the canvas and move on.
- Keep the running canvas visible: after every exchange, show the canvas with what is known so far, so the user always sees where the interview stands.
- Concretize vague answers before banking them: ask for the role affected, the current state, a number, a deadline, or the decision point.
- Blank means unknown, not skippable. Every unknown goes to **Open Questions** with the next piece of evidence that would settle it.

Grilling supplies the rest of the interview discipline: facts are the agent's job, decisions are the user's, each round asks the whole frontier with a recommended answer per question.

## The stages

Work the stages in order: each round's frontier holds only the active stage's questions. If the user volunteers an answer from a later stage, bank it on the canvas and keep it locked there until its own stage. In Stages 0 to 4, a volunteered solution is banked as context and translated back into the pain it was meant to treat; no solution branch is asked until the gate.

### Stage 0. Requirement intake

Stay on the _pain_. What hurts, who hurts, how they cope today, and why today's way fails. Done when you can state the problem in one non-technical sentence with a number or a named moment in it, e.g. "Sales spend 15 minutes finding a suitable route per shipment."

### Stage 1. Outcome discovery

Rewrite "do a project" into an observable change in the business world. Whose work or whose customer experience changes? Which of speed, cost, conversion, reliability, risk, or compliance improves, by how much, by when? Done when you have a business outcome with target values, a deadline, and at least one **guardrail**: a counter-metric that must not regress (e.g. search time down, route acceptance rate flat).

### Stage 2. Decision and behavior analysis

Name the specific decision or behavior that must be supported or changed. What does the user or stakeholder ultimately decide? What do they judge on today, with nothing new in hand? What does a wrong call cost? Done when one decision or behavior statement stands alone, e.g. "which route fits this shipment and this customer?"

### Stage 3. Output and deliverable design

Specify the deliverable as something that can be produced and checked: Given which inputs, output what (a value, probability, category, ranking, document, report, automated action, or explanation)? Who consumes it, how, and how will anyone tell it worked? Done when you can write the spec as `Given <inputs> → Output <deliverable>`, with its consumer and its check named.

### Stage 4. Reality check

Face the data and the constraints. Historical labels or outcome data? Freshness, coverage, gaps, errors? Access and who approves it? When do inputs arrive, and when is the decision due? Organizational, budget, and compliance limits? Dependent teams, systems, and external factors? Done when the canvas carries a data-and-constraint assessment and every blocker has a named next action (e.g. data not ready, so first event tracking, consistent IDs, and data contracts).

### Stage 5. Baseline and complexity gate

Prove the cheap options fall short before complexity enters. What do rules, process change, a better UI, a report, or plain automation achieve against the Stage 1 targets? Where exactly do they start to break? Does the marginal gain of added complexity beat its cost and risk? Done when you have a stated _baseline_ and a reasoned Go / No-Go: the baseline meets the targets, ship it and stop here; it falls short, name where, and proceed.

### Stage 6. Solution and intervention design

Reached only through the gate. Pick the intervention type (rule, process, UI, automation, analysis, model, policy), define the minimum viable version, and say who reviews and corrects its results, how it rolls out, and what gets monitored. Done when the canvas carries intervention type, minimum viable scope, reviewer, and rollout-and-learning plan.

## The canvas

Write the finished canvas to `problem-definition-<slug>.md` in the current directory (slug from the topic) and report the path. Blank fields keep their blank-with-next-evidence form from Open Questions: never paper over an unknown to complete the document.

Closing handoff: report the path and point the user at `/grill-with-docs` as the next step. The canvas defines the problem; `/grill-with-docs` sharpens it into a buildable idea. It feeds the thinking, it doesn't replace it.

<canvas-template>

# Problem Definition Canvas: <title>

**Problem:** one sentence, non-technical.

**Target User / Stakeholder:** who is affected, who uses the output, who takes part in the change.

**Current Workflow:** how the work happens today, and where it jams.

**Business Outcome:** the observable change in the real world.

**Success Metrics + Guardrails:** targets with values and deadlines, plus the counter-metrics that must not regress.

**Decision or Behavior:** the one decision or behavior being supported or changed.

**Ideal Output / Deliverable:** `Given <inputs> → Output <deliverable>`, plus its consumer and its check.

**Data and Information Available:** labels, outcomes, freshness, coverage, quality, access, governance.

**Constraints and Timing:** when inputs arrive, decision latency, uptime, explainability, org, budget, compliance.

**Baseline and Non-Complex Options:** what rules, process, UI, reports, or plain automation achieve, and where they break.

**Recommended Solution and Intervention Design:** intervention type, minimum viable scope, review cadence.

**Review and Feedback:** who corrects errors, how feedback flows in, where explanations surface.

**Rollout and Learning Plan:** pilot, ramp, monitoring, alerts, solution health, iteration.

**Work Type:** product feature, process improvement, policy change, data project, automation, experiment, or AI/ML.

**Open Questions / Assumptions:** every blank, each with the next evidence that would settle it.

</canvas-template>
