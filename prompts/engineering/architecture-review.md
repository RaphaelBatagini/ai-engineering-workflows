# Architecture review prompt

> **Category:** engineering
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/engineering/architecture-review.md`](../../workflows/engineering/architecture-review.md)

---

## Purpose

Produces a structured tradeoff analysis for an architecture proposal: strengths, risks, operational implications, and comparison against stated alternatives, so the engineering team has a structured basis for the decision.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `problem_statement` | Required | What the system cannot currently do, or what new requirement exists. Specific and measurable if possible. |
| `constraints` | Required | Performance requirements, operational constraints, team capability, timeline, cost. What cannot be changed. |
| `options` | Required | At least two options with brief descriptions. Include the proposed/preferred option. |
| `proposed_approach` | Required | The preferred design in enough detail to analyze: component interactions, data flows, key technology choices. |
| `current_state` | Optional | Brief description of what currently exists and what this proposal changes from. Helps contextualize the tradeoffs. |
| `evaluation_criteria` | Optional | Specific dimensions to prioritize in the analysis: "prioritize operational simplicity," "optimize for consistency over availability," "team has limited Go experience." |

---

## Prompt

```
You are producing a structured tradeoff analysis for an architecture proposal. Your output supports the engineering team's decision — it does not make the decision.

Problem statement:
[PLACEHOLDER: paste the problem statement]

Constraints:
[PLACEHOLDER: paste the constraints]

Options under consideration:
[PLACEHOLDER: paste the options with brief descriptions]

Proposed approach (detail):
[PLACEHOLDER: paste the proposed approach with component interactions and key technology choices]

Current state (if provided):
[PLACEHOLDER: paste current state description, or omit]

Evaluation criteria priority (if specified):
[PLACEHOLDER: paste evaluation_criteria, or omit]

---

Produce the following analysis:

**1. Proposed approach analysis**
- Strengths: what this approach does well against the stated constraints and requirements
- Risks: specific failure modes, operational complexity, scaling limitations, or maintenance burden
- Operational implications: deployment complexity, observability requirements, runbook needs, failure recovery
- Scalability and longevity: how this ages as load or team structure changes

**2. Alternative comparison**
For each alternative stated above: how it compares to the proposed approach on the key dimensions. Be specific — "Alternative B has lower operational complexity because X, but introduces Y limitation."

**3. Assumptions requiring validation**
What assumptions in the proposal need to be confirmed before this decision is finalized? List specific things to verify — not generic engineering concerns.

**4. Questions not addressed**
What important questions does the proposal leave open that the team should resolve before accepting it?

Do not recommend a final choice — that is the engineering team's decision. If the stated constraints make one option clearly better, say so and explain specifically why based on the constraints.
```

---

## Output contract

1. **Proposed approach analysis:** Strengths, risks, operational implications, scalability assessment
2. **Alternative comparison:** Specific comparison per alternative against key dimensions
3. **Assumptions requiring validation:** Specific, enumerated assumptions to verify
4. **Questions not addressed:** Open questions with enough context to know who should resolve them

---

## Validation criteria

- [ ] Risk analysis is specific — named failure modes, not "this could be complex"
- [ ] Alternative comparisons reference the stated constraints — not generic preferences
- [ ] Assumptions are specific and actionable — not "validate that this will scale"
- [ ] The analysis does not advocate for a choice without grounding in the stated constraints

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Generic analysis | Risks apply to any distributed system, not this proposal | Add `evaluation_criteria` and more detail in `proposed_approach` |
| Missing operational analysis | Deployment, observability, and runbook implications are absent | Add to prompt: "Explicitly address operational implications including deployment, on-call burden, and observability" |
| Advocating strongly for one option | Analysis reads as a recommendation disguised as analysis | Remind the model: "Do not recommend a final choice" |
| Shallow alternative analysis | Alternatives dismissed in one sentence | Expand `options` section with more detail per alternative |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
