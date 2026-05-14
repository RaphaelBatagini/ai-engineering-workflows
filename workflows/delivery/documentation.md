# Documentation workflow

> **Category:** delivery
> **Typical duration:** 20–45 minutes
> **Tools:** Claude Code or Claude.ai
> **Related prompt:** [`prompts/delivery/documentation.md`](../../prompts/delivery/documentation.md)

---

## Context

This workflow structures AI-assisted generation of engineering documentation: service READMEs, runbooks, API documentation, architecture overviews, ADRs, and onboarding guides. AI produces a structured first draft from provided context; the engineer validates accuracy, fills gaps from tacit knowledge, and owns the final document.

The key constraint is accuracy. AI-generated documentation that contains inaccuracies is worse than missing documentation — it misleads confidently. Every factual claim in the output must be validated before publication.

---

## When to use this workflow

- Writing a README or service overview for an existing service with no documentation
- Creating a runbook for an operational procedure
- Generating API documentation from code and specifications
- Producing an architecture overview document from design notes or ADRs
- Creating onboarding documentation for a service or team

## When not to use this workflow

- Documentation that requires extensive tacit knowledge that cannot be expressed in a prompt (the manual approach is more reliable)
- Documenting decisions not yet made — finalize the decision first

---

## Prerequisites

- [ ] The documentation type and audience are defined
- [ ] The source material is assembled: code, ADRs, design docs, runbooks, tickets, or other authoritative references
- [ ] You have enough system knowledge to validate accuracy of the output

---

## Steps

### Step 1: Define documentation scope and audience

**AI role:** None
**Human role:** Define what to document and for whom

Specify before running analysis:
- **Document type:** README, runbook, API docs, architecture overview, onboarding guide
- **Audience:** New engineers onboarding, on-call engineers, external consumers, tech leads
- **Scope:** What the document covers and what it explicitly does not cover
- **Source material:** What authoritative references the model should use

The audience definition drives appropriate depth, assumed knowledge level, and tone. A runbook for on-call engineers looks very different from an architecture overview for new team members.

> **Validation gate:** Scope, audience, and source material are defined before the first prompt is run.

---

### Step 2: Assemble source material

**AI role:** None
**Human role:** Collect the authoritative sources the documentation will be derived from

Assemble:
- Relevant source code (key interfaces, entry points, configuration structures)
- Existing ADRs or design documents
- Existing partial documentation
- Runbook steps or operational procedures (if documenting operations)
- API specifications or OpenAPI definitions

The model generates documentation from provided context. If the context is incomplete, the model will fill gaps with plausible-sounding content that may be inaccurate. Providing comprehensive source material is the primary lever for documentation quality.

> **Validation gate:** Source material is assembled and covers the scope of the document to be written.

---

### Step 3: Generate the draft

**AI role:** Produce a structured first draft from the provided source material, audience definition, and document scope
**Human role:** Provide the assembled context; review the draft for structure and completeness

Use the [documentation prompt](../../prompts/delivery/documentation.md) with the assembled context.

Review the draft for structure first:
- Does it cover the intended scope?
- Is it organized appropriately for the audience?
- Are there sections that are clearly missing?

Do not review for accuracy yet — that is the next step.

> **Validation gate:** The draft covers the intended scope and is structured appropriately for the audience.

---

### Step 4: Validate for accuracy

**AI role:** None
**Human role:** Verify every factual claim in the draft against authoritative sources

This is the most critical step. Work through the draft systematically:

- Every technical claim is verifiable against source code, ADRs, or running system behavior
- No features, endpoints, or behaviors are described that don't exist
- Configuration examples are correct and would actually work
- Command-line examples run successfully
- No contradictions with existing authoritative documentation

Mark every claim you cannot verify. Resolve each one by consulting the source or removing the claim.

> **Validation gate:** Every factual claim in the document is either verified or removed. "It seems right" is not a validation outcome.

---

### Step 5: Fill gaps and finalize

**AI role:** Optional — can expand specific sections given additional context
**Human role:** Fill gaps from tacit knowledge; finalize and publish

Add to the draft:
- Context that exists in your head but wasn't in the source material
- Operational notes, common pitfalls, and debugging tips that come from experience
- Links to related documents, tickets, and runbooks

The final document should reflect what an experienced team member would write — not just what can be inferred from the source code.

---

## Output format

Published documentation in the appropriate location: repository README, docs directory, Confluence, or equivalent. Format follows the team's existing documentation standards.

---

## Quality checklist

Before publishing:

- [ ] Every technical claim is verified — no unverified assertions
- [ ] No hallucinated features, APIs, or behaviors
- [ ] Configuration and command examples are tested and correct
- [ ] The document reads naturally for the target audience — not over-qualified or under-explained
- [ ] Links and references point to current, correct locations
- [ ] The document is consistent with existing documentation in style and terminology

---

## Known limitations

- **Tacit knowledge gaps:** The model can document what is explicitly in the source material. Operational wisdom, common failure modes, and non-obvious system behaviors that exist only in engineers' heads require manual addition.
- **Accuracy risk:** This is the highest-risk output type in this repository. Inaccurate documentation misleads confidently. Validation in Step 4 is not optional.
- **Freshness:** AI-generated documentation reflects the state of the source material at the time of generation. Documentation requires the same maintenance discipline as code — it goes stale.

---

## Related workflows

- [Onboarding](onboarding.md) — for onboarding-specific documentation and materials
- [Architecture Review](../engineering/architecture-review.md) — produces the ADR that often serves as source material for documentation
