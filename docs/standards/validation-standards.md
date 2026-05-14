# Validation Standards

This document defines the review gates and approval criteria for AI-assisted outputs across all workflow types.

---

## What validation is

Validation is the process by which an engineer confirms that an AI-assisted output meets the quality criteria for its type and is appropriate to use. Validation is not proofreading. It is not a quick scan. It is structured review against explicit criteria.

Validation is always done by a human engineer. It cannot be delegated to another AI pass, automated tests alone, or a process that treats AI output as equivalent to engineer-authored output. This is not because AI outputs are inherently worse — it is because the engineer is accountable for the final artifact, and accountability requires genuine review.

---

## Validation levels

### Level 1: Self-validation

The engineer who ran the workflow validates the output before use.

**Requirements:**
- Work through the relevant [quality gates checklist](../../guardrails/quality-gates.md)
- Apply the [review checklist](../../guardrails/review-checklist.md)
- Confirm the human judgment gate: "I can stand behind this output as my own professional judgment"

**Appropriate for:** Async communication drafts, ticket refinement outputs, documentation drafts, debugging hypothesis lists, routine PR review analyses.

---

### Level 2: Peer validation

The output is reviewed by a second engineer before use.

**Requirements:**
- Level 1 validation must be completed first
- The reviewing engineer receives: the original prompt, the full output, and the first engineer's validation notes
- The reviewer applies the relevant quality gates independently
- Disagreements are resolved before the output is accepted

**Appropriate for:** Architecture review outputs, technical decision records, postmortem root cause conclusions, incident impact assessments intended for cross-team or external distribution.

---

### Level 3: Structured review

The output goes through a defined review process with sign-off from appropriate stakeholders.

**Requirements:**
- Level 2 validation must be completed first
- Reviewers are defined before review begins — not determined after
- Each reviewer validates against the output's quality criteria, not just reads and approves
- Sign-off is explicit and traceable (PR approval, documented in the artifact, tracked comment)

**Appropriate for:** Public-facing documentation, security-relevant design decisions, postmortem reports distributed beyond the engineering team, infrastructure change documentation.

---

## Validation timing

Validation must happen before the output is used — not after. This means:

- Before code is merged
- Before documentation is published
- Before incident conclusions are communicated
- Before a ticket is committed to a sprint

Retroactive validation (reviewing after the output has been acted on) is not a valid alternative. If timeline pressure prevents proper validation, the answer is to reduce scope — not reduce review rigor.

---

## The role of system knowledge in validation

Engineers validating AI outputs are responsible for applying three layers:

1. **Quality gates** — the explicit criteria for the output type
2. **System knowledge** — things the model cannot know: organizational context, unwritten constraints, historical decisions, team conventions, operational realities
3. **Implication assessment** — what acting on this output would mean in practice, not just whether the output is internally consistent

System knowledge is a required input to validation. An engineer who lacks the domain knowledge to evaluate an output should not validate it alone — they should pair with someone who has the context. Validating an output you cannot actually evaluate is not validation.

---

## Documenting validation outcomes

For Level 2 and Level 3 validations, document:

- What was validated, by whom, and when
- Any criteria that failed and how they were resolved
- Any limitations or caveats that future consumers of the artifact should know

This documentation lives with the artifact — in the PR, the ADR, the postmortem document — not in a separate tracking system.

---

## Validation failure handling

When validation fails:

1. **Identify the failure type** — accuracy, completeness, scope violation, calibration, hallucination
2. **Determine the cause** — insufficient prompt context, bad prompt structure, inherent model limitation for this task type, wrong workflow for the task
3. **Fix the cause, not just the output** — if a prompt consistently produces a bad output pattern, update the prompt
4. **Document the failure mode** — add it to the prompt's failure modes section if it's not already there

Do not silently correct AI output and move on without addressing the root cause. Unresolved prompt issues produce the same failures across all engineers using the workflow.
