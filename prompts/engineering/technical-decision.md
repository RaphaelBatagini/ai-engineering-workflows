# Technical decision prompt

> **Category:** engineering
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/engineering/technical-decision.md`](../../workflows/engineering/technical-decision.md)

---

## Purpose

Produces a structured tradeoff analysis for a scoped technical decision. The output surfaces the assumptions embedded in each option, the risks that are hardest to see when close to a problem, and the questions that remain open — giving the engineer a more complete picture before deciding.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `decision_statement` | Required | A single, specific decision framed as "Should we X or Y?" or "Which approach should we take for Z?" Vague framing produces vague analysis. |
| `options` | Required | At least two candidate approaches. For each: a brief description, and the key known tradeoffs. |
| `constraints` | Required | Non-negotiable constraints (latency budget, team size, existing stack, timeline). Options that violate constraints should be flagged, not analyzed in depth. |
| `evaluation_criteria` | Optional | What matters most for this decision (e.g., operational simplicity, scalability to X, time-to-implement). If omitted, the model will infer from context — verify what was inferred. |
| `current_state` | Optional | Relevant details about the existing system this decision affects. Helps the model identify migration complexity and integration risk. |
| `prior_decisions` | Optional | Related ADRs or past decisions that constrain the option space or establish relevant precedent. |

---

## Prompt

```
You are supporting a technical decision analysis. Your role is to surface tradeoffs, assumptions, and open questions — not to recommend a specific option. The engineer will make the final decision.

**Decision statement:**
[decision_statement]

**Options under consideration:**
[options]

**Constraints:**
[constraints]

**Evaluation criteria:**
[evaluation_criteria — or "Not specified; infer from context and state your inference explicitly"]

**Current system context:**
[current_state — or "Not provided"]

**Prior related decisions:**
[prior_decisions — or "None provided"]

---

Produce a structured tradeoff analysis with the following sections:

**1. Constraint check**
For each option: does it satisfy all stated constraints? Flag any option that violates a constraint and explain why. Do not analyze constraint-violating options further unless asked.

**2. Per-option analysis**
For each viable option:
- Core tradeoffs: what you gain and what you give up
- Key assumptions this option relies on (what must be true for this to work well)
- Operational implications: what changes about how the system is run, monitored, or scaled
- Failure mode: what goes wrong if the core assumptions are incorrect

**3. Comparative assessment**
A direct comparison across the viable options against the evaluation criteria. Identify where the options diverge most significantly. Do not aggregate into a recommendation.

**4. Open questions and validation gaps**
What needs to be true or verified before this decision can be made with confidence? Include: unknowns that would change the analysis, assumptions that are hard to validate upfront, and risks that only surface in production.

Format: structured sections with clear headers. No summary recommendation. No preference statements.
```

---

## Output contract

The model should return four sections:

1. **Constraint check** — each option assessed against stated constraints; non-viable options flagged
2. **Per-option analysis** — core tradeoffs, assumptions, operational implications, and failure mode for each viable option
3. **Comparative assessment** — direct comparison across evaluation criteria without a recommendation
4. **Open questions and validation gaps** — unknowns that must be resolved before deciding

The output should be factual and structured. Any preference language ("Option A is better", "clearly the right choice") is a failure mode.

---

## Validation criteria

- [ ] Constraint check correctly identifies which options are viable given the stated constraints
- [ ] Each option's assumptions are made explicit — not left as unstated premises
- [ ] Operational implications are concrete, not generic ("requires monitoring" is too vague; "requires alerting on queue depth because the consumer is now async" is acceptable)
- [ ] The comparative assessment uses the stated evaluation criteria, not invented ones
- [ ] No recommendation is made — output is structured analysis, not a conclusion
- [ ] Open questions are specific and actionable, not generic ("test in staging" without specifying what to test)

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Preference language | Output says "clearly better", "recommend", "obviously", or similar | Reject and re-run with explicit instruction: "Do not recommend. State tradeoffs only." |
| Missing assumption surfacing | Analysis describes tradeoffs but doesn't identify what must be true for each option to succeed | Re-run with: "For each option, explicitly state: what assumptions must hold for this option to work as described?" |
| Generic operational analysis | "Requires monitoring and alerting" without specifics | Provide more system context and re-run: "Be specific about what metrics, failure signals, and operational procedures each option requires." |
| Constraint-violating option analyzed | Output analyzes an option that violates a stated constraint | Flag the error; re-run with constraints emphasized: "Option [X] violates constraint [Y]. Do not include it in the comparative analysis." |
| Invented evaluation criteria | Output assesses options against criteria not in the input | Compare output criteria against input criteria; discard analysis sections that use invented criteria |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
