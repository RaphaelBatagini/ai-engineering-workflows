# Ticket refinement workflow

> **Category:** delivery
> **Typical duration:** 10–25 minutes
> **Tools:** Claude Code or Claude.ai; Jira (direct or via MCP)
> **Related prompt:** [`prompts/delivery/ticket-refinement.md`](../../prompts/delivery/ticket-refinement.md)

---

## Context

This workflow uses AI to improve the completeness and testability of tickets before sprint commitment. It surfaces missing acceptance criteria, ambiguous requirements, edge cases, and scope boundary issues that are commonly absent from first-draft tickets.

The output prepares tickets for better-informed refinement discussions — reducing mid-sprint scope clarification, improving story point accuracy, and reducing the rate of tickets returned from code review for scope issues.

---

## When to use this workflow

- Backlog refinement sessions where tickets will be committed to a sprint
- Pre-grooming of high-priority tickets before a refinement meeting
- Feature tickets with complex acceptance criteria or cross-service impact
- Tickets authored by non-technical stakeholders that need technical scoping

## When not to use this workflow

- Bug tickets with a clear reproduction path and explicit acceptance criteria
- Spike tickets where the output is research, not a defined deliverable
- Trivial chores with no functional acceptance criteria

---

## Prerequisites

- [ ] The ticket has a description — even if incomplete
- [ ] You understand the product context: what problem this solves and for whom
- [ ] You have enough system knowledge to evaluate whether proposed acceptance criteria are accurate

---

## Steps

### Step 1: Assess the ticket as-written

**AI role:** None
**Human role:** Evaluate the ticket before running AI analysis

Before running analysis, assess:
- Does the ticket have a problem statement, or just a solution description?
- Are acceptance criteria present? Are they testable?
- Is scope bounded? Is it clear what is in and what is out?
- Are there obvious edge cases, dependencies, or integration points missing?

This baseline assessment lets you evaluate the model's output rather than receiving it uncritically.

> **Validation gate:** You understand what the ticket is asking before sending it to the model. Do not send an unread ticket to the model.

---

### Step 2: Run AI-assisted ticket analysis

**AI role:** Identify missing acceptance criteria, ambiguous requirements, edge cases, and scope risks
**Human role:** Provide ticket content and product context; prepare to evaluate output

Use the [ticket refinement prompt](../../prompts/delivery/ticket-refinement.md) with the ticket content.

Expected output:
- Proposed acceptance criteria in testable format
- Identified ambiguities with clarifying questions
- Edge cases not addressed in the ticket
- Scope observations: what appears in vs. out of scope
- Dependencies or integration points that may be missing

> **Validation gate:** Review every item against product and technical context. The model generates plausible-sounding criteria but cannot know product decisions, team conventions, or technical constraints that aren't in the ticket.

---

### Step 3: Curate the analysis output

**AI role:** None
**Human role:** Apply product and technical judgment to every proposed addition

For each proposed acceptance criterion:
- Is it testable? (Can pass/fail be determined unambiguously?)
- Is it accurate to the intended behavior — not just plausible?
- Does it reflect product decisions that have actually been made?

For each identified ambiguity:
- Is this genuinely ambiguous, or does the team have an implicit answer?
- Who needs to resolve it — PM, tech lead, stakeholder?

For each edge case:
- Is this in scope for this ticket, or intentionally deferred?
- Is it a real edge case in this system, or a generic concern?

Remove, modify, or accept each item. Do not copy the model's output directly into the ticket — curate it first.

> **Validation gate:** Every addition to the ticket is something you can defend in a refinement meeting. Nothing is added solely because the model suggested it.

---

### Step 4: Update the ticket

**AI role:** Optional — can help phrase acceptance criteria in a consistent format
**Human role:** Update the ticket; verify completeness post-update

Apply the curated additions to the ticket. Verify after updating:
- All acceptance criteria are testable (binary pass/fail, no subjective terms)
- Scope is explicitly bounded
- Edge cases are addressed or explicitly noted as deferred with a reason
- Ambiguities are resolved or escalated with an identified owner

> **Validation gate:** A developer picking up this ticket cold can begin work without a synchronous clarification conversation.

---

### Step 5: Team review in refinement

**AI role:** None
**Human role:** Walk the team through the refined ticket

For complex tickets, walk through the refined acceptance criteria with the engineering team in the refinement meeting. Engineers will surface implementation-level edge cases and technical constraints that neither the PM nor the AI analysis identified.

This step is the highest-value source of ticket improvement. AI analysis prepares the ticket for this conversation — it does not replace it.

---

## Output format

An updated ticket with:
- Testable acceptance criteria
- Explicit scope boundary
- Documented edge cases (in scope or explicitly deferred)
- Resolved or escalated ambiguities

---

## Quality checklist

Before committing the ticket to a sprint:

- [ ] Every acceptance criterion uses verifiable language — no "fast," "seamless," "user-friendly," "handles gracefully"
- [ ] Scope boundary is explicit — what is in and what is out is stated
- [ ] Edge cases are addressed or explicitly deferred with a reason
- [ ] A developer can start work without a clarification conversation
- [ ] Story points were estimated against the refined scope, not the original scope

---

## Known limitations

- **Product context gaps:** The model cannot know product decisions, user research findings, or stakeholder agreements that exist outside the ticket. These must be applied by the PM or the engineer doing the refinement.
- **Technical constraint blindness:** Model-suggested edge cases may not reflect real system constraints. Technical reviewers must validate them.
- **Over-specification risk:** The model sometimes proposes more acceptance criteria than are practical or necessary. Curating down to the necessary set is part of the workflow — exhaustive criteria create maintenance burden.

---

## Related workflows

- [Documentation](documentation.md) — for generating technical documentation from refined tickets
- [Technical Decision](../engineering/technical-decision.md) — when a ticket surfaces a technical choice that warrants formal documentation
