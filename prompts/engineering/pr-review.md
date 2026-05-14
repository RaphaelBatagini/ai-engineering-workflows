# PR review prompt

> **Category:** engineering
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/engineering/pr-review.md`](../../workflows/engineering/pr-review.md)

---

## Purpose

Produces a structured PR review analysis covering logic correctness, edge cases, security considerations, scope alignment, and test coverage for the reviewing engineer to validate and act on.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `diff` | Required | The full PR diff. Paste the raw diff output or provide via GitHub MCP. Large diffs should be split by logical section. |
| `pr_description` | Required | PR title and description as written by the author. |
| `ticket_context` | Required | Linked ticket: title, problem statement, and acceptance criteria. Scope alignment cannot be assessed without this. |
| `service_context` | Optional | Brief description of the service: purpose, criticality, technology stack, and relevant constraints (e.g., "public API with versioning constraints," "critical path for payment processing"). |
| `review_focus` | Optional | Specific areas to prioritize: "focus on error handling," "security-first review," "check for performance regression." Constrains the analysis when the diff is broad. |

---

## Prompt

```
You are producing a technical PR review analysis. Your output is a structured first pass that the reviewing engineer will validate and act on — not a substitute for their review.

Service context (if provided):
[PLACEHOLDER: paste service_context here, or omit this section]

Linked ticket (problem statement and acceptance criteria):
[PLACEHOLDER: paste ticket title, problem statement, and acceptance criteria]

PR description:
[PLACEHOLDER: paste PR title and description]

PR diff:
[PLACEHOLDER: paste the full diff]

Review focus (if specified):
[PLACEHOLDER: paste review_focus, or omit this section]

---

Produce a structured analysis with the following sections. For each finding, state: the specific file and function, a concise description of the issue, and severity (blocking | non-blocking).

**1. Scope alignment**
Does the diff implement what the ticket describes? Are there acceptance criteria unaddressed? Are there changes in scope beyond what the ticket specifies?

**2. Logic and correctness**
Are there logic errors, incorrect assumptions, or conditions under which the code produces incorrect results? Be specific — name the function, the condition, and the expected vs. actual behavior.

**3. Edge cases**
What input conditions, concurrency scenarios, or state transitions are not handled? Prioritize edge cases with non-obvious failure consequences.

**4. Security considerations**
Are there input validation gaps, authorization issues, credential exposure risks, injection vulnerabilities, or unsafe data handling introduced by this diff? Only flag issues directly introduced — do not produce a general security audit.

**5. Error handling**
Are errors handled at appropriate levels? Are error messages appropriate (not leaking internal state)? Are retries or fallbacks present where the system design requires them?

**6. Test coverage**
Are the changes covered by tests? Are tests validating behavior or implementation details? Are edge cases from section 3 represented in tests?

**7. Observations** (optional, clearly labeled non-blocking)
Low-priority style or pattern observations. Label each explicitly as non-blocking.

If you cannot assess a section based on the provided context, say so explicitly. Do not generate generic observations as placeholders.
```

---

## Output contract

Expected response structure:

1. **Scope alignment:** Specific gaps or over-scoping, with ticket criterion references
2. **Logic and correctness:** Specific issues with file/function location and severity label
3. **Edge cases:** Unhandled conditions with location and severity
4. **Security considerations:** Security issues specific to this diff with location and severity
5. **Error handling:** Gaps with location and severity
6. **Test coverage:** Coverage gaps or test quality issues
7. **Observations:** Non-blocking items, clearly labeled

---

## Validation criteria

- [ ] Every finding references a specific file and function — no generic observations
- [ ] Severity labels are present on every finding (blocking / non-blocking)
- [ ] Scope alignment section references the actual ticket acceptance criteria
- [ ] Security section is specific to the diff — not a general security checklist
- [ ] Observations are clearly distinguished from blocking findings

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Generic analysis | Observations apply to any codebase ("consider adding error handling," "this may have performance issues") | Add `service_context` and `review_focus` to constrain the scope |
| Missing scope alignment | Scope section is empty or says "changes appear to match the ticket" without specific analysis | Confirm that acceptance criteria are included in `ticket_context` input |
| Over-flagging | 15+ findings, many low-signal | Add `review_focus` to constrain; manually triage and discard low-signal items |
| Hallucinated functions | Model references functions or patterns not present in the diff | Remove the item; verify the model is not drawing from stale or irrelevant context |
| Security theater | Security section lists generic vulnerabilities not relevant to this diff | Revise prompt to add: "Only flag security issues directly introduced by this diff" |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
