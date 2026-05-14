# Onboarding prompt

> **Category:** delivery
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/delivery/onboarding.md`](../../workflows/delivery/onboarding.md)

---

## Purpose

Generates onboarding material for a specific service or codebase — either a service orientation document, a getting started runbook, or a codebase exploration guide — from provided source material. The output requires engineer validation and completion with tacit knowledge before it is used with a new engineer.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `output_type` | Required | What to generate. Options: `service-orientation`, `getting-started-runbook`, `codebase-guide`. Each produces a differently structured output. |
| `new_engineer_context` | Required | Brief description of the incoming engineer's background and role (e.g., "senior backend engineer, familiar with Go, new to this service area", "junior engineer, first production system"). Calibrates assumed knowledge and level of detail. |
| `source_material` | Required | Source to synthesize from: README files, ADRs, runbook notes, architecture diagrams, existing documentation. More source material produces better output and fewer placeholders. |
| `service_name` | Required | The name of the service or codebase being documented. |
| `critical_paths` | Optional | The most operationally significant code paths (e.g., "the payment processing path", "the authentication flow"). The orientation document will emphasize these. |
| `known_gotchas` | Optional | Known sharp edges, non-obvious behaviors, or common mistakes for new engineers. If provided, these will be included explicitly — these are the items least likely to appear in source documentation. |

---

## Prompt

```
You are generating onboarding material for an engineer joining or taking ownership of a service. Your role is to synthesize source material into a structured, accurate starting point. The engineer preparing these materials will validate every claim, add tacit knowledge, and complete all gaps before use.

**Output type:**
[output_type]

**Service name:**
[service_name]

**New engineer context:**
[new_engineer_context]

**Source material:**
[source_material]

**Critical paths to emphasize:**
[critical_paths — or "Not specified; infer from source material and state your inference"]

**Known gotchas to include:**
[known_gotchas — or "None provided; do not fabricate"]

---

Generate the requested onboarding material following the structure below.

**For service-orientation:**
1. What this service does and why it exists (one paragraph, no jargon)
2. How it fits into the broader system — what calls it and what it calls
3. The critical paths and their operational characteristics
4. Common failure modes and how to recognize them
5. Key design decisions (from ADRs or source material)
6. Where to go for more information

**For getting-started-runbook:**
1. Prerequisites (access, tools, environment)
2. Setting up the development environment (numbered steps with exact commands)
3. Running the service locally (numbered steps)
4. Running the tests
5. Making and deploying a small change (the canonical first PR)
6. Accessing staging and production

**For codebase-guide:**
1. Repository structure — what each top-level directory contains
2. Entry points — where the service starts and how requests flow in
3. Key modules — what each major module does and how they interact
4. Data model overview
5. How to find your way around (where to look for X)

For all output types: where source material is absent or insufficient, insert a placeholder:
> [NEEDS ENGINEER INPUT: what's missing and why it matters for the new engineer]

Do not fabricate commands, configuration, or technical details. Flag inferred claims:
> [VERIFY: this is inferred from [source]; confirm before use]

Write for the stated engineer context. Do not include meta-commentary.
```

---

## Output contract

The model should return a structured markdown document matching the selected output type, written for the stated engineer context. The document should:
- Follow the structure defined for the output type
- Include `[NEEDS ENGINEER INPUT: ...]` placeholders where source is insufficient
- Include `[VERIFY: ...]` flags on inferred technical claims
- Use exact commands from source material — never invent commands
- Contain no meta-commentary

---

## Validation criteria

- [ ] Every command in the getting-started runbook has been executed and produces the described output
- [ ] The service orientation produces an accurate mental model — verifiable by having someone unfamiliar with the service read it
- [ ] Technical claims are traceable to source material or explicitly flagged
- [ ] Known gotchas are included (if provided) — not omitted as "non-standard"
- [ ] The level of detail is appropriate for the stated engineer context

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Fabricated commands | Setup or deployment commands in the output that don't appear in source material and aren't flagged | Reject; every command in a getting-started runbook must come from source material. Re-run with: "Do not include any command not present in the source material. Insert [NEEDS ENGINEER INPUT] instead." |
| Missed gotchas | Known sharp edges provided in input don't appear in the output | Check output against provided `known_gotchas`; re-run with explicit instruction to include each one |
| Over-generic service orientation | The orientation could describe any service — it doesn't explain what makes this service specific | Re-run with more source material; add: "Be specific to this service. Generic statements about microservices or REST APIs add no value." |
| Wrong knowledge level | Output written at the wrong level for the stated engineer context | Re-run with explicit calibration: "The reader [has/has never] worked with [technology]. Adjust depth accordingly." |
| Missing entry point analysis | Codebase guide doesn't identify where the service starts and how requests flow | Re-run with: "Identify the main entry point and trace the flow for the most common request type from entry point to response." |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
