# Debugging workflow

> **Category:** engineering
> **Typical duration:** 20–60 minutes
> **Tools:** Claude Code or Claude.ai; access to logs, stack traces, and relevant source code
> **Related prompt:** [`prompts/engineering/debugging.md`](../../prompts/engineering/debugging.md)

---

## Context

This workflow structures AI-assisted debugging sessions for bugs that are not immediately obvious — where the cause is unclear, symptoms are ambiguous, the codebase area is unfamiliar, or multiple potential causes need systematic triage.

AI contributes structured hypothesis generation: it processes the symptoms, stack traces, and source context to produce a ranked candidate list with confirmation strategies for each. The engineer applies system knowledge to triage those hypotheses and leads the investigation. The root cause conclusion is always engineer-confirmed — not model-asserted.

---

## When to use this workflow

- The bug cause is not immediately apparent from reading the code or error message
- Symptoms are ambiguous or inconsistent across reproduction attempts
- The failing component is in unfamiliar code
- Multiple potential causes exist and need structured triage
- An intermittent issue requires methodical investigation rather than trial-and-error patching

## When not to use this workflow

- The bug and fix are immediately obvious — use standard debugging
- A reproduction path is confirmed and root cause is already known — proceed to fix design
- The issue has a known runbook with a deterministic resolution path — follow the runbook

---

## Prerequisites

- [ ] The observed vs. expected behavior is documented in concrete terms
- [ ] Reproduction steps are known, or intermittency is characterized (frequency, conditions)
- [ ] Relevant logs, stack traces, or error messages are available
- [ ] The affected component or service is identified

---

## Steps

### Step 1: Document the failure precisely

**AI role:** None
**Human role:** Write a precise failure description before involving AI

Before running any analysis, write a structured failure description:

```
Observed behavior: [exact description — not "it's broken" but "endpoint X returns HTTP 500 when Y is true"]
Expected behavior: [exact description]
Reproduction steps: [numbered list]
Frequency: [always | intermittent at ~X% | triggered by specific condition]
First observed: [timestamp or deployment]
Recent changes: [deployments, config changes, dependency updates in the last 48h]
Environment: [production | staging | local | which region/cluster]
```

The quality of the failure description directly determines the quality of the hypothesis output. Precision here is the highest-leverage step in the workflow.

> **Validation gate:** You can describe the failure to someone unfamiliar with the system and they understand exactly what is broken and under what conditions.

---

### Step 2: Assemble investigation context

**AI role:** None
**Human role:** Collect the raw artifacts that will ground the hypothesis analysis

Collect:
1. Stack trace or error message — exact text, not paraphrased
2. Relevant log lines with timestamps from the failure window
3. Source code of the failing component — the relevant functions and their immediate call context, not the full file
4. Recent changes in the affected area (git log output, relevant diff)
5. Monitoring data if relevant: error rate timeline, latency spike, resource metrics

Provide raw artifacts to the model, not your interpretation of them. The model generates better hypotheses from a real stack trace than from a description of what the stack trace shows.

> **Validation gate:** You have actual artifacts assembled — not summaries. If a stack trace is unavailable, note its absence explicitly rather than paraphrasing what you remember.

---

### Step 3: Generate hypotheses

**AI role:** Generate a ranked list of root cause candidates with supporting evidence, contradicting evidence, and confirmation strategies
**Human role:** Provide the failure description and artifacts; prepare to evaluate the output

Use the [debugging prompt](../../prompts/engineering/debugging.md) with the assembled context.

Expected output:
- Ranked hypothesis list (typically 3–6 candidates)
- For each: supporting evidence from the provided artifacts, evidence that would contradict it, and a specific confirmation test
- Any additional context that would improve hypothesis quality if the analysis is inconclusive

> **Validation gate:** Review every hypothesis against your system knowledge. Apply what the model cannot know: service topology, known technical debt, historical failure patterns, recent operational changes. Add hypotheses the model missed. Remove hypotheses incompatible with system facts you know. Do not proceed with the model's ranking unchanged.

---

### Step 4: Triage hypotheses

**AI role:** None
**Human role:** Prioritize investigation based on validated hypothesis list and system knowledge

From the validated hypothesis list, select the top 1–3 hypotheses to investigate first, weighted by:
- Likelihood given the full symptom picture
- Ease of confirmation or elimination (fast eliminations narrow the field quickly)
- Impact if correct (prioritize high-blast-radius root causes for earlier confirmation)

Write down your triage decision and the rationale. If you deprioritize a hypothesis, note why — this prevents revisiting it without new evidence.

> **Validation gate:** You have a written, prioritized list of 1–3 hypotheses with reasoning for the ordering before beginning active investigation.

---

### Step 5: Investigate systematically

**AI role:** Assist with specific investigation mechanics — crafting log queries, analyzing a code path, suggesting test cases, interpreting unfamiliar stack frames
**Human role:** Execute investigation; document findings; update hypothesis confidence

For each hypothesis in priority order:

1. Define what evidence would confirm or rule it out
2. Gather that evidence (run the query, add a log line, write a test, trace the code path)
3. Document the finding — confirming or disconfirming — before moving to the next hypothesis
4. Update confidence accordingly

Use AI to help with mechanics — crafting a regex for a log query, explaining what a specific Go runtime error means, suggesting edge case test inputs. Do not use AI to decide whether a hypothesis is confirmed.

> **Validation gate:** Before concluding a hypothesis is confirmed, you have direct evidence — not just absence of evidence against it. A hypothesis "consistent with symptoms" is not confirmed; it is a leading candidate.

---

### Step 6: Confirm root cause and design the fix

**AI role:** Assist with fix implementation, edge case analysis for the fix, and regression test suggestions
**Human role:** State the root cause; own the fix design

Once a hypothesis is confirmed by evidence:

1. State the root cause precisely: what code path, under what conditions, produces what incorrect behavior
2. Design the fix — AI can assist with implementation, but validate every change against the root cause statement
3. Verify: does this fix address the root cause, or only mask the symptom?
4. Identify regression tests that would catch this class of failure in future

> **Validation gate:** You can explain the root cause in one paragraph to a colleague without referencing the AI analysis. If you cannot explain it, you have not confirmed it — you have accepted a plausible-sounding hypothesis.

---

## Output format

A debugging session produces:
1. A root cause statement (precise causal description)
2. A code fix
3. Regression test coverage for the failure mode
4. A brief incident note if the bug was production-impacting

---

## Quality checklist

Before closing the investigation:

- [ ] Root cause is specific: named component, specific code path, specific triggering condition — not a category
- [ ] The fix addresses the root cause, not just the symptom
- [ ] Regression coverage exists that would catch this failure mode in CI
- [ ] Related code paths that share the same pattern have been audited
- [ ] If production-impacting: an incident note or Jira comment documents the root cause and fix

---

## Known limitations

- **Intermittent failures:** Hypothesis generation works well for deterministic failures. For intermittent issues, the model generates candidates but cannot prioritize by frequency — apply your observation history to weight the triage.
- **Deep system knowledge:** The model cannot know your service's full topology, team conventions, or historical failure patterns. Triage (Step 4) must be performed by someone with domain context.
- **Context window and large codebases:** Provide only the relevant code path, not entire files. Extraneous context degrades hypothesis quality by adding noise.
- **Novel libraries or internal frameworks:** The model may not have current training data on recently released libraries or proprietary frameworks. Validate any framework-specific hypotheses independently.

---

## Related workflows

- [Incident Investigation](incident-investigation.md) — for production incidents where scope management and stakeholder communication are additional concerns
- [PR Review](pr-review.md) — for reviewing the fix PR after root cause is confirmed
