# Architecture review workflow

> **Category:** engineering
> **Typical duration:** 30–90 minutes
> **Tools:** Claude Code or Claude.ai; relevant design documents
> **Related prompt:** [`prompts/engineering/architecture-review.md`](../../prompts/engineering/architecture-review.md)

---

## Context

This workflow structures AI-assisted architecture reviews for proposed system changes, new service designs, significant refactors, or technology selections. AI accelerates tradeoff analysis and surfaces considerations that may be overlooked — it does not make the decision or own the conclusions.

This workflow is appropriate for decisions that create lasting constraints on the system: data model changes, service boundary decisions, technology selections, integration patterns, and scalability design.

---

## When to use this workflow

- Evaluating a proposed architecture for a new service or major feature
- Reviewing a significant refactor or migration design
- Comparing multiple design options with non-obvious tradeoffs
- Structuring a technical decision that will bind future choices and warrant an ADR

## When not to use this workflow

- Implementation decisions that don't create lasting architectural constraints
- Decisions already made that just need documentation — use the [ADR template](../../templates/adr-template.md) directly
- Exploratory spikes — design first, review after you have a concrete proposal

---

## Prerequisites

- [ ] The proposal is written, not just a verbal description or Slack conversation
- [ ] The problem being solved is explicitly stated
- [ ] Constraints are documented: performance requirements, operational constraints, team capability, timeline, cost
- [ ] At least two alternatives have been considered and briefly described
- [ ] The current system state (what this changes from) is described

---

## Steps

### Step 1: Verify proposal completeness

**AI role:** Optional — can help structure an informal proposal into a reviewable format
**Human role:** Confirm the proposal meets minimum completeness criteria

A reviewable architecture proposal requires:
- **Problem statement:** What is the system failing to do, or what new capability is required?
- **Constraints:** Performance, operational, organizational, timeline, cost
- **Options evaluated:** At least two, with brief rationale for each
- **Proposed approach:** The preferred design with initial rationale
- **Open questions:** Known unknowns that haven't been resolved

If the proposal lacks these sections, it is not ready for review. If you use AI to structure an informal proposal into this format, verify that the content is accurate — the model will fill gaps with plausible-sounding content that may not reflect the actual situation.

> **Validation gate:** The proposal is complete per the criteria above. A review of an incomplete proposal wastes time and produces low-signal analysis.

---

### Step 2: Run tradeoff analysis

**AI role:** Generate a structured tradeoff analysis covering the proposed approach, alternatives, operational implications, and known failure modes
**Human role:** Provide the complete proposal; prepare to evaluate output against system context

Use the [architecture review prompt](../../prompts/engineering/architecture-review.md) with the full proposal.

Expected output:
- Analysis of the proposed approach: strengths, risks, operational complexity, known failure modes
- Comparative analysis against the alternatives
- Operational considerations: deployment complexity, observability requirements, failure modes, runbook implications
- Scalability and maintenance outlook
- Questions or assumptions in the proposal that need validation

> **Validation gate:** Review every item against your knowledge of the system, team, and operational context. The model cannot know your team's operational maturity, on-call coverage, existing runbook depth, or real production load patterns. These factors materially affect architectural decisions and must be applied explicitly in Step 3.

---

### Step 3: Identify gaps and validate assumptions

**AI role:** Can help enumerate questions or identify missing analysis from the tradeoff output
**Human role:** Curate the gap list against actual system and organizational context

From the tradeoff analysis, identify:
- Assumptions in the proposal that need explicit validation before the decision is finalized
- Missing analysis (failure modes not considered, dependencies not mapped, migration path not addressed)
- Questions that must be answered to choose responsibly between options
- Organizational or operational context that changes the tradeoff calculus

Remove gaps that are not real given the system context. Add gaps the model couldn't identify from the proposal alone. The curated gap list becomes the agenda for review discussions and the open questions section of the ADR.

> **Validation gate:** The gap list is curated — not the raw model output. You have applied your system knowledge to distinguish real gaps from model-generated noise.

---

### Step 4: Make and document the decision

**AI role:** Assist with drafting the Architecture Decision Record from the analysis, discussions, and resolved gaps
**Human role:** Make the decision; author and own the ADR

Use the model to draft the [ADR](../../templates/adr-template.md) from:
- The complete proposal
- The tradeoff analysis
- The resolved gaps from Step 3
- The decision and its rationale

Review the draft ADR carefully:
- Is the problem statement accurate and specific?
- Is the decision unambiguous — could two readers interpret it differently?
- Are the consequences honest, including the negative ones?
- Are the alternatives described fairly — would a proponent of each alternative recognize their option?
- Are open questions captured with owners?

> **Validation gate:** The ADR is peer-reviewed by at least one engineer with relevant system context (Level 2 validation per [validation-standards.md](../../docs/standards/validation-standards.md)). The decision-maker signs off on the final document before it is considered accepted.

---

## Output format

An Architecture Decision Record in `docs/decisions/` following the [ADR template](../../templates/adr-template.md), with status set to `Accepted`.

---

## Quality checklist

Before marking the ADR as accepted:

- [ ] Problem statement is specific — not "we need better scalability" but a measurable gap
- [ ] The decision is unambiguous — readable by someone in 18 months without clarification
- [ ] Negative consequences are documented honestly — every option has costs
- [ ] Alternatives are described accurately — proponents of each would recognize their option
- [ ] Open questions have owners and target dates
- [ ] At least one peer has reviewed the ADR with relevant system knowledge

---

## Known limitations

- **Operational realism:** The model can analyze architectural patterns and theoretical failure modes. It cannot assess your team's actual operational maturity, on-call load, or incident response capability. These factors materially affect many architectural decisions and must be applied by the engineer.
- **Organizational context:** Team structure, ownership boundaries, and inter-team constraints are not visible to the model. Engineer judgment is required.
- **Novel patterns:** AI-assisted tradeoff analysis covers well-established architectural patterns. Genuinely novel system designs may have failure modes or tradeoffs not in the model's training distribution.
- **Long-term accuracy:** Architectural decisions often involve predictions about future load, team growth, or technology trajectory. The model can structure the reasoning but cannot predict these outcomes more accurately than the engineer.

---

## Related workflows

- [Technical Decision](technical-decision.md) — for narrower technical decisions that don't require a full architectural review
- [PR Review](pr-review.md) — for reviewing implementation PRs that follow from an architecture decision
- [Documentation](../delivery/documentation.md) — for generating technical documentation based on a finalized architecture
