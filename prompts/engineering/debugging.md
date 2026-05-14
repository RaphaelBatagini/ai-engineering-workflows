# Debugging prompt

> **Category:** engineering
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/engineering/debugging.md`](../../workflows/engineering/debugging.md)

---

## Purpose

Generates a ranked list of root cause hypotheses from a structured failure description and observability artifacts, with evidence assessment and confirmation strategies for each hypothesis.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `failure_description` | Required | Structured description: observed behavior, expected behavior, reproduction steps, frequency, first observed, recent changes, environment. See Step 1 of the debugging workflow. |
| `stack_trace` | Required* | Exact stack trace or error message. *Required if available — if not available, note its absence explicitly. |
| `log_excerpts` | Required* | Relevant log lines from the failure window with timestamps. *Required if available. |
| `source_context` | Required | The relevant source code: the failing function(s) and their immediate call context. Not the full file — focused on the failure path. |
| `recent_changes` | Optional | Recent commits or deployments in the affected area: git log output, relevant diff summary. |
| `system_context` | Optional | Service description: language, runtime, framework, key dependencies relevant to the failure, known technical debt or known failure patterns. |

---

## Prompt

```
You are assisting with a structured debugging investigation. Based on the provided context, generate a ranked list of root cause hypotheses. Ground every hypothesis in the provided evidence.

System context (if provided):
[PLACEHOLDER: paste system_context, or omit this section]

Failure description:
[PLACEHOLDER: paste the structured failure description]

Stack trace / error message:
[PLACEHOLDER: paste the exact stack trace or error message — or note "not available"]

Log excerpts (with timestamps):
[PLACEHOLDER: paste the relevant log lines]

Source code context:
[PLACEHOLDER: paste the relevant source code — the failing function and its immediate context]

Recent changes (if provided):
[PLACEHOLDER: paste recent git log output or deployment notes, or omit]

---

Produce the following:

**Hypotheses** (ranked by likelihood based on the provided evidence):

For each hypothesis, provide:
- **Hypothesis:** A specific causal statement. Not "a memory leak" — "the connection pool is not releasing connections in the error path of function X because rows.Close() is not deferred."
- **Supporting evidence:** What in the provided context supports this hypothesis. Reference specific log lines, stack frames, or source locations.
- **Contradicting evidence:** What in the provided context argues against this hypothesis, or what would rule it out.
- **Confirmation test:** A specific, executable action that would confirm this hypothesis — a log query, a code change to add instrumentation, a test to run.
- **Elimination test:** A specific, executable action that would rule this hypothesis out.

**Missing context** (if applicable):
If the provided context is insufficient to generate confident hypotheses, specify what additional information would most improve the analysis. Be concrete: name the specific log source, metric, or code path.

Do not speculate beyond the evidence. If you cannot connect a hypothesis to the provided context, do not include it.
```

---

## Output contract

1. **Hypotheses:** Ranked list (typically 3–6) with supporting/contradicting evidence and specific confirmation/elimination tests per hypothesis
2. **Missing context:** Specific additional information that would improve the analysis (if the provided context is insufficient)

---

## Validation criteria

- [ ] Each hypothesis is a specific causal statement — not a category of issues
- [ ] Confirmation tests are executable — concrete, named, specific actions
- [ ] Supporting evidence is grounded in the provided artifacts — not in general patterns that could apply to any codebase
- [ ] The ranking reflects likelihood based on the evidence — not alphabetical or arbitrary

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Vague hypotheses | "Memory leak," "race condition," "database issue" without specifics | Provide more precise source code context and failure description; add `system_context` |
| Untethered hypotheses | Hypotheses not connected to provided artifacts (no evidence citations) | Provide raw stack trace and log excerpts rather than descriptions of them |
| Over-speculation | Hypotheses for scenarios not supported by any evidence | Append to prompt: "Only include hypotheses directly supported by the provided context" |
| Context window noise | Many hypotheses are about code not related to the failing path | Provide only the relevant source path — not full files |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
