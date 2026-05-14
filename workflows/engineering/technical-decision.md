# Technical decision workflow

> **Category:** engineering
> **Typical duration:** 20–45 minutes
> **Tools:** Claude Code or Claude.ai
> **Related prompt:** [`prompts/engineering/technical-decision.md`](../../prompts/engineering/technical-decision.md)

---

## Context

This workflow structures AI-assisted technical decisions: library selection, API design choices, implementation pattern selection, and other technical choices that have meaningful consequences but don't require a full architectural review.

AI contributes structured tradeoff analysis for the options under consideration. The engineer applies system context, team context, and operational judgment to reach the decision. For decisions with broader architectural implications, use the [Architecture Review workflow](architecture-review.md) instead.

---

## When to use this workflow

- Selecting between competing libraries or dependencies
- Choosing between multiple valid implementation patterns
- Making an API design decision with non-obvious long-term implications
- Any technical choice that warrants documentation beyond a PR comment
- Decisions that other engineers will need to understand and be consistent with

## When not to use this workflow

- Obvious technical choices with a clear right answer — decide and move on
- Decisions that require broader architectural analysis — use [Architecture Review](architecture-review.md)
- Implementation details that don't create consistency or maintenance implications

---

## Prerequisites

- [ ] The decision to be made is clearly stated
- [ ] The options being considered are identified (at least two)
- [ ] The evaluation criteria are identified: what makes one option better than another for this context?
- [ ] Relevant constraints are known: performance requirements, team familiarity, existing patterns in the codebase, compatibility requirements

---

## Steps

### Step 1: Frame the decision

**AI role:** None
**Human role:** Write a concise decision frame before seeking analysis

Write a decision frame that includes:
- The specific question to be answered (one sentence)
- The options under consideration with brief descriptions
- The evaluation criteria in priority order
- The constraints that eliminate options

Framing the decision before running analysis prevents the model from substituting a different question. It also forces clarity on what you actually need to decide.

> **Validation gate:** The decision frame is specific enough that two engineers reading it would agree on what is being decided.

---

### Step 2: Run tradeoff analysis

**AI role:** Generate structured tradeoff analysis for each option against the defined criteria
**Human role:** Provide the decision frame; evaluate output against system and team context

Use the [technical decision prompt](../../prompts/engineering/technical-decision.md) with the decision frame.

Expected output:
- Per-option analysis against each stated criterion
- Risks and downsides for each option (including the favored one)
- Operational considerations (maintenance, observability, debugging experience)
- Recommendation with rationale, qualified by what context the model doesn't have

> **Validation gate:** Review every claim against your knowledge of the codebase, team, and operational context. Apply what the model cannot know: existing precedent in the codebase, team familiarity, real performance characteristics, organizational constraints.

---

### Step 3: Decide and document

**AI role:** Can assist with drafting a decision record or ADR from the analysis
**Human role:** Make the decision; own the rationale

Make the decision. Document it at the appropriate level:
- **PR comment or commit message:** For routine implementation decisions with no lasting cross-team implications
- **README or CONTRIBUTING note:** For conventions that other engineers need to follow
- **ADR:** For decisions that will bind future choices or need to be referenced over time

If documenting as an ADR, use the [ADR template](../../templates/adr-template.md).

> **Validation gate:** The decision is documented in a place where affected engineers will find it. An undocumented technical decision is a future maintenance burden.

---

## Output format

Decision documentation at the appropriate level: PR comment, codebase note, or ADR.

---

## Quality checklist

- [ ] The decision is specific — not "use X" but "use X for Y purpose, not for Z"
- [ ] The rationale is recorded — not just what was decided but why
- [ ] The tradeoffs are honest — the negatives of the chosen option are acknowledged
- [ ] The decision is documented where affected engineers will encounter it

---

## Known limitations

- **Codebase specificity:** The model's analysis is based on the options and criteria you provide. It cannot know what patterns already exist in your codebase that would make one option more or less consistent with established conventions.
- **Benchmark accuracy:** For performance-sensitive decisions, the model can describe known characteristics of options but cannot benchmark them in your specific environment. Validate performance claims empirically for high-stakes decisions.

---

## Related workflows

- [Architecture Review](architecture-review.md) — for decisions with broader system-level implications
- [PR Review](pr-review.md) — for reviewing the implementation that follows from the decision
