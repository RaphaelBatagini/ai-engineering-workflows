# Onboarding workflow

> **Category:** delivery
> **Typical duration:** 30–60 minutes (preparation); ongoing during onboarding period
> **Tools:** Claude Code or Claude.ai
> **Related prompt:** [`prompts/delivery/onboarding.md`](../../prompts/delivery/onboarding.md)

---

## Context

This workflow structures AI-assisted preparation of onboarding materials for new engineers joining a team or service. AI accelerates the generation of service overviews, codebase orientation guides, and "day 1" runbooks from existing documentation and source context. Engineers validate accuracy and add the tacit knowledge that documentation alone cannot capture.

This workflow also covers using AI during the onboarding period itself: new engineers can use AI-assisted analysis sessions to explore unfamiliar code systematically rather than through trial and error.

---

## When to use this workflow

- Preparing onboarding materials before a new engineer joins
- Generating a service orientation guide for an engineer taking ownership of a new area
- New engineer accelerating codebase familiarization during the first weeks

## When not to use this workflow

- As a substitute for pairing time with an experienced team member — AI-assisted onboarding materials supplement human knowledge transfer, they do not replace it
- Generating documentation about systems that are not yet documented enough to validate the output — close the documentation gap first

---

## Prerequisites

**For onboarding material preparation:**
- [ ] The service has enough existing documentation to provide as source material (README, ADRs, runbooks)
- [ ] You have system knowledge sufficient to validate the accuracy of generated materials
- [ ] The new engineer's starting role is known (their responsibilities affect what to emphasize)

**For new engineer codebase exploration:**
- [ ] Basic development environment is set up
- [ ] Access to the codebase is confirmed

---

## Steps (onboarding material preparation)

### Step 1: Identify knowledge transfer scope

**AI role:** None
**Human role:** Define what the new engineer needs to know in week 1 vs. month 1

Identify:
- **Day 1:** Environment setup, key contacts, communication channels, incident response basics
- **Week 1:** Service overview, codebase orientation, deployment process, on-call basics
- **Month 1:** Design decisions (ADRs), operational runbooks, cross-team integrations, technical debt landscape

Scoping the materials prevents producing a comprehensive documentation dump that's too dense to absorb in the first days.

> **Validation gate:** Scope and priority order are defined before generating any materials.

---

### Step 2: Generate service overview

**AI role:** Produce a service orientation document from source code, ADRs, and existing documentation
**Human role:** Provide source material; validate accuracy; add tacit knowledge

Use the [documentation workflow](documentation.md) to generate the service overview. Specifically for onboarding, emphasize:
- What this service does and why it exists
- How it fits into the broader system (what calls it, what it calls)
- The critical paths and their operational characteristics
- Common failure modes and how to recognize them

After generation, add the tacit knowledge that documentation alone doesn't capture:
- Which parts of the codebase are the most fragile and why
- Common gotchas that trip up new engineers
- The history behind significant design decisions

> **Validation gate:** The service overview is accurate. Every technical claim is verified. A new engineer reading it would form a correct mental model.

---

### Step 3: Generate "getting started" runbook

**AI role:** Structure a step-by-step getting started guide from setup scripts, READMEs, and onboarding notes
**Human role:** Validate every step; run through the guide in a fresh environment if possible

The getting started runbook covers:
- Development environment setup
- Running the service locally
- Running the tests
- Making and deploying a small change (the canonical first PR)
- Accessing staging and production environments

Validate every command in the runbook. Broken setup steps on day 1 create a poor first impression and waste the new engineer's time disproportionately.

> **Validation gate:** Every command in the runbook has been executed and produces the described output.

---

### Step 4: Prepare a first-week task list

**AI role:** Optional — can suggest a structured first-week task sequence from the onboarding scope
**Human role:** Define the actual first task set based on team context and the engineer's background

The first task list should balance:
- Getting familiar with the codebase through small, real changes
- Understanding the deployment and review process
- Making a meaningful contribution early

> **Validation gate:** First-week tasks are achievable in the first week with the level of context the new engineer will have.

---

## Steps (new engineer codebase exploration)

### Step 5: AI-assisted codebase orientation

**AI role:** Explain code areas, trace call paths, identify key entry points, summarize design patterns
**Human role:** Direct the exploration; validate explanations against observed behavior

New engineers can use AI-assisted sessions to:
- Understand what a specific module does and how it fits into the broader service
- Trace the execution path for a specific request type
- Understand the rationale for a design pattern (especially useful for patterns documented in ADRs)

When using AI for codebase exploration, treat explanations as starting points to verify — not authoritative answers. The model may misunderstand internal patterns or make assumptions about code organization that don't match your specific codebase.

---

## Output format

- Service orientation document
- Getting started runbook
- First-week task list

---

## Quality checklist

- [ ] Every command in the getting started runbook executes successfully
- [ ] The service overview produces an accurate mental model (verified by having someone unfamiliar with the service read it)
- [ ] Tacit knowledge — gotchas, fragile areas, history — is included, not just what's in the source
- [ ] First-week tasks are calibrated to be achievable without excessive unblocking

---

## Known limitations

- **Tacit knowledge gap:** This is the highest limitation of this workflow. The most valuable onboarding knowledge — why decisions were made, which parts of the system are fragile, where the bodies are buried — lives in engineers' heads, not in source code. AI can generate a structural orientation, but human pairing is required to transfer operational wisdom.
- **Validation burden:** Onboarding materials must be validated more carefully than most other AI-generated output because errors in them are repeated across every new engineer who uses them.

---

## Related workflows

- [Documentation](documentation.md) — for the documentation generation steps in this workflow
- [Code Review](../engineering/code-review.md) — useful for new engineers reviewing an unfamiliar service area
