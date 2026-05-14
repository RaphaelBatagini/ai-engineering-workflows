# Code review workflow

> **Category:** engineering
> **Typical duration:** 20–45 minutes
> **Tools:** Claude Code or Claude.ai
> **Related prompt:** [`prompts/engineering/code-review.md`](../../prompts/engineering/code-review.md)

---

## Context

This workflow structures AI-assisted systematic code quality reviews — deeper reviews not tied to a specific PR, used for quality assessments of a module, feature area, or recently onboarded codebase. It complements the [PR Review workflow](pr-review.md) which is scoped to a specific change.

Use this workflow for: code quality assessments before a major refactor, reviewing a service you're taking ownership of, onboarding-level codebase familiarization with structured output, or systematic identification of technical debt in a module.

---

## When to use this workflow

- Taking ownership of an unfamiliar service or module and needing a structured quality assessment
- Preparing for a major refactor: understanding the current state before defining the target state
- Conducting a systematic technical debt review in a specific area
- Code quality review as part of a team health or architecture check

## When not to use this workflow

- Reviewing a specific pull request — use [PR Review](pr-review.md)
- General codebase exploration without a defined review goal

---

## Prerequisites

- [ ] The scope of the review is defined: which module, service, or file set
- [ ] The review goal is stated: what kind of issues are you looking for?
- [ ] You have access to the relevant source code
- [ ] You have a basic understanding of the service's purpose and operational context

---

## Steps

### Step 1: Define review scope and criteria

**AI role:** None
**Human role:** Define what will be reviewed and against what criteria

Before running analysis, specify:
- **Scope:** Which files, modules, or directories
- **Focus areas:** Logic correctness, error handling, security, test coverage, performance, maintainability — pick the most relevant 2–4
- **Context:** Service purpose, criticality, known constraints, team conventions

The model produces better-targeted analysis when the review focus is explicit. "Review this code" produces generic output. "Review this payment processing module for error handling completeness and security boundary enforcement" produces specific, actionable analysis.

> **Validation gate:** Review scope and focus areas are written down before the first prompt is run.

---

### Step 2: Run structured analysis

**AI role:** Produce a structured quality analysis of the specified code against the defined focus areas
**Human role:** Provide the code and review criteria; evaluate output against system context

Use the [code review prompt](../../prompts/engineering/code-review.md) with the scoped code and focus areas.

For large codebases, review one module or logical section at a time. The model's analysis quality degrades with large, noisy context — focused input produces focused output.

> **Validation gate:** Review every finding against your system knowledge. Apply operational context, team conventions, and known constraints the model cannot access.

---

### Step 3: Categorize and prioritize findings

**AI role:** None
**Human role:** Triage findings by severity and action type

Organize the validated findings:

| Category | Description |
|---|---|
| **Defects** | Logic errors, incorrect behavior — fix before next release |
| **Security** | Vulnerabilities, unsafe patterns — fix with priority |
| **Reliability** | Missing error handling, untested failure modes — fix in near term |
| **Technical debt** | Maintainability issues, pattern inconsistencies — track and address systematically |
| **Observations** | Low-priority notes — log but don't necessarily act on immediately |

> **Validation gate:** Every finding in the Defects and Security categories has a clear action and owner.

---

### Step 4: Document findings and action plan

**AI role:** Can assist with structuring the review output document
**Human role:** Author the findings document and action plan

Produce a review summary with:
- Prioritized finding list with severity labels
- Recommended actions with priority order
- Items to track as technical debt

For findings that warrant immediate action, create tickets or PRs directly from the review. For technical debt items, add to the team's debt backlog with context.

---

## Output format

A review document with categorized findings, severity labels, and a prioritized action plan.

---

## Quality checklist

- [ ] Every finding is specific: file, function, and the exact issue
- [ ] Severity is accurate — high-severity items are genuinely high-risk, not inflated
- [ ] Defects and Security findings have clear owners and action timelines
- [ ] Technical debt items are tracked in the team's backlog, not left in the review document
- [ ] Findings reflect the codebase reviewed, not generic patterns

---

## Known limitations

- **Context window for large modules:** Large code sections need to be reviewed in logical chunks. The model may miss interactions between chunks. Identify cross-module concerns manually.
- **Correctness vs. intent:** The model can identify what the code does and flag deviations from common patterns. It cannot know what the code was intended to do if that intent isn't documented or inferable.

---

## Related workflows

- [PR Review](pr-review.md) — for reviewing a specific change rather than a code area
- [Architecture Review](architecture-review.md) — when the code review surfaces design-level concerns
