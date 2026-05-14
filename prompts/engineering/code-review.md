# Code review prompt

> **Category:** engineering
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/engineering/code-review.md`](../../workflows/engineering/code-review.md)

---

## Purpose

Produces a structured analysis of a code area or changeset for defects, security issues, reliability concerns, and technical debt. Intended as a structured first-pass that the engineer validates and categorizes before acting on findings.

This prompt is for systematic code review of an area or module — distinct from the PR review prompt, which is scoped to a specific proposed change.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `code` | Required | The code to review. Paste the full source, or provide file references if using Claude Code with filesystem access. |
| `language_and_framework` | Required | The language and relevant frameworks (e.g., "Go 1.22, standard library only" or "TypeScript, Express, Prisma"). Affects idiom and security analysis. |
| `review_focus` | Required | What this review is looking for. Options: `defects`, `security`, `reliability`, `tech-debt`, `all`. Scoped reviews produce more useful output than unfocused ones. |
| `system_context` | Optional | What this code does in the broader system. Helps the model identify risks that are only visible with operational context (e.g., "this is the hot path called 10k req/s"). |
| `known_constraints` | Optional | Known constraints that affect how findings should be interpreted (e.g., "we can't change the external API contract", "this is read-only code"). |

---

## Prompt

```
You are performing a structured code review. Your role is to identify issues in the categories specified. The engineer will validate findings and decide which to address.

**Code under review:**
[code]

**Language and framework:**
[language_and_framework]

**Review focus:**
[review_focus]

**System context:**
[system_context — or "Not provided"]

**Known constraints:**
[known_constraints — or "None stated"]

---

Produce a structured review with findings organized by category. Only include categories relevant to the specified review focus.

**Defects**
Logic errors, incorrect assumptions, off-by-one errors, incorrect error handling, data corruption risks. For each finding: location (line or function), description of the defect, and the condition under which it manifests.

**Security**
Authentication/authorization gaps, injection risks, insecure handling of secrets or credentials, unvalidated external input, information exposure. For each finding: location, vulnerability type, and the attack surface it creates.

**Reliability**
Unhandled error paths, missing retries on transient failures, improper resource management (connections, file handles, goroutines, memory), timeout handling, race conditions. For each finding: location, failure mode, and the condition under which it triggers.

**Technical debt**
Code that works but creates maintenance risk: unclear naming that obscures intent, duplication that will need to stay in sync, patterns inconsistent with the rest of the codebase, missing tests for critical paths. For each finding: location, description, and the specific future risk it creates.

**Observations**
Minor style inconsistencies, non-idiomatic patterns, or low-priority improvements that do not warrant a finding category. Group together at the end. These are informational only.

Format findings as a numbered list within each category. Do not mix categories. If a category has no findings, state "None identified" — do not omit the section.
```

---

## Output contract

The model should return findings organized into the five sections: Defects, Security, Reliability, Technical Debt, Observations. Each finding includes:
- Location (line number or function name)
- Description of the issue
- The condition under which it manifests or the risk it creates

Sections with no findings should state "None identified." The model should not combine categories or omit sections from the review focus.

---

## Validation criteria

- [ ] Every finding has a specific location — no findings that say "generally" or "throughout the code"
- [ ] Every defect finding includes the condition under which it manifests, not just a description
- [ ] Security findings identify the specific attack surface, not just the vulnerability type
- [ ] Reliability findings identify the specific failure condition (e.g., "when the database connection times out" not "in error cases")
- [ ] Technical debt findings explain the future risk, not just that the code is "messy"
- [ ] No findings contradict the stated known constraints
- [ ] Observations section does not contain items that belong in the finding categories

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Superficial security analysis | Security findings list "input validation" or "SQL injection" without identifying where in the code and under what condition | Provide more context about the input surface and re-run with: "Be specific about which inputs are unvalidated and what the attack vector is." |
| Hallucinated references | Finding references a function, line, or variable that doesn't exist in the provided code | Cross-reference every finding location against the source; discard or re-query hallucinated findings |
| Over-flagging observations as findings | Observations-level items (style, naming) appear as Defects or Reliability findings | Reclassify during validation; add a constraint to the prompt: "Findings must represent an actual defect or risk, not a style preference." |
| Missing error path analysis | No reliability findings in code that has many error paths, suggesting the model skipped them | Re-run with focus: "Review specifically for unhandled error paths and missing resource cleanup." |
| Generic tech debt findings | "This function is too long" without explaining why it creates future risk | Reject and ask: "What specific maintenance risk does this create? What breaks if it's not addressed?" |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
