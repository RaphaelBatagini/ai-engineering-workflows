# Ticket refinement prompt

> **Category:** delivery
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/delivery/ticket-refinement.md`](../../workflows/delivery/ticket-refinement.md)

---

## Purpose

Analyzes a ticket and produces: proposed acceptance criteria in testable format, identified ambiguities with clarifying questions, edge cases not addressed, and scope observations.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `ticket_content` | Required | The full ticket: title, description, any existing acceptance criteria, and relevant comments. |
| `product_context` | Optional | Brief description of the feature area, user context, or product decisions already made. Helps prevent criteria that contradict product intent. |
| `system_context` | Optional | Relevant technical context: affected service, data model, integration points, known constraints. Helps surface technically relevant edge cases. |
| `scope_boundaries` | Optional | Explicit statements of what is and is not in scope for this ticket. Prevents the model from generating criteria for out-of-scope functionality. |

---

## Prompt

```
You are helping to refine an engineering ticket for sprint readiness. Your output is input to the refinement discussion — the engineer and PM will validate every item before updating the ticket.

Product context (if provided):
[PLACEHOLDER: paste product_context, or omit]

System context (if provided):
[PLACEHOLDER: paste system_context, or omit]

Scope boundaries (if specified):
[PLACEHOLDER: paste scope_boundaries, or omit]

Ticket content:
[PLACEHOLDER: paste the full ticket — title, description, existing acceptance criteria, relevant comments]

---

Produce the following:

**1. Acceptance criteria**
Proposed acceptance criteria in testable format. Each criterion must be:
- Binary: it either passes or fails — no subjective terms ("good performance," "user-friendly," "handles errors gracefully")
- Specific: it describes a concrete, verifiable behavior
- Scoped: it applies to what this ticket describes, not adjacent functionality

**2. Ambiguities**
Questions the ticket leaves unanswered that would prevent a developer from starting work or completing it without a clarification conversation. For each: state the ambiguous element and the clarifying question.

**3. Edge cases**
Input conditions, state transitions, or user scenarios not addressed by the current ticket description. For each edge case: describe it and note whether it appears in or out of scope based on the ticket.

**4. Scope observations**
Is the ticket scope well-bounded? If not, what appears unclear or unbounded? What appears potentially out of scope based on the description?

Do not generate acceptance criteria for functionality not described in the ticket. If the ticket is too vague to generate specific criteria, state that and describe what additional information is needed.
```

---

## Output contract

1. **Acceptance criteria:** Testable, specific, scoped criteria
2. **Ambiguities:** Specific questions with context
3. **Edge cases:** Named edge cases with in/out-of-scope assessment
4. **Scope observations:** Scope boundary analysis

---

## Validation criteria

- [ ] Every acceptance criterion uses only verifiable language — no "fast," "reliable," "intuitive," "graceful"
- [ ] Acceptance criteria match the ticket's described functionality — no criteria for out-of-scope features
- [ ] Ambiguities are specific — they identify what exactly is unclear, not just "the requirements are vague"
- [ ] Edge cases are relevant to the described system, not generic patterns

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Over-specification | 15+ acceptance criteria for a routine feature | Add `scope_boundaries`; manually curate the output down to necessary criteria |
| Subjective criteria | Criteria contain "fast," "user-friendly," "appropriate," "sufficient" | Revise each criterion to use measurable language before adding to ticket |
| Out-of-scope criteria | Criteria describe functionality not in the ticket | Apply product context to filter; add explicit `scope_boundaries` |
| Generic edge cases | Edge cases apply to any web application ("what if the user is offline?") | Add `system_context` to ground edge cases in the actual system |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
