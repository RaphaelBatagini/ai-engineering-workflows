# Output Quality Standards

This document defines what constitutes an acceptable AI-assisted output in engineering workflows.

---

## The fundamental question

Before accepting any AI-assisted output, ask:

> "Can I stand behind this output as my own professional judgment?"

If the answer is no — if you could not defend every claim from first principles — the output is not ready. Revise the prompt, run additional validation, or produce the output manually.

---

## Quality dimensions

### Accuracy

The output must be factually correct based on the context provided.

- Technical claims are verifiable
- No hallucinated APIs, methods, services, or behaviors
- Numbers, thresholds, and metrics are not fabricated
- References to external systems are accurate

Accuracy is the highest-priority quality dimension. An output that is complete, well-structured, and confidently wrong is worse than an output that is incomplete and accurate.

### Specificity

Generic outputs are low-value. An output should be specific to the system, codebase, incident, or decision at hand — not a rephrasing of general engineering principles that apply to any project.

Signals of low specificity:
- "This code may have performance issues depending on load"
- "Consider adding error handling"
- "This architecture could face scalability challenges at scale"

These are not actionable. A high-quality output names the specific function, the specific missing error case, the specific failure mode at a specific condition.

### Completeness

The output should cover the full scope defined in the output contract. Silent omissions are a common failure mode — the model produces a response that looks complete but drops sections or edge cases without flagging them.

Check: does the output address everything that was asked? Is anything from the input context suspiciously absent from the output?

### Scope adherence

The output should not go beyond what was asked. Unrequested scope expansion introduces noise, increases review time, and can propagate errors in areas outside the engineer's current focus.

Scope expansion signals:
- Refactoring recommendations when the prompt asked for a bug analysis
- Architecture suggestions when the prompt asked for a documentation draft
- General security audit when the prompt asked for a PR review

### Calibration

Confidence levels in the output should match the evidence. "Confirmed root cause" requires direct evidence. "Probable cause" requires supporting signals. "Possible cause" is a hypothesis.

Watch for overconfident language when evidence is ambiguous — and for excessive hedging when evidence is clear. Both are miscalibration.

---

## Output acceptance criteria by type

See [guardrails/quality-gates.md](../../guardrails/quality-gates.md) for per-type checklists covering:
- Code outputs
- Analysis outputs (code review, architecture review, debugging)
- Documentation outputs
- Investigation outputs (incident, DataDog, debugging)
- Communication outputs
- Ticket and requirement outputs

---

## When to reject an output

**Reject and revise the prompt when:**
- The output contains factual errors about the system
- The output is generic — it could apply to any similar context
- Critical sections of the output contract are missing
- The output goes significantly beyond the requested scope
- Confidence claims are clearly miscalibrated

**Reject and produce manually when:**
- Three prompt revisions have not improved quality meaningfully
- The required context cannot be expressed in a prompt (too much tacit knowledge, too complex a system state)
- The risk of a bad output being accepted is high enough that manual production is the more reliable path

---

## Quality trend tracking

When a prompt consistently produces outputs requiring significant correction, the failure mode should be documented in the prompt's failure modes section. This benefits all engineers using the same workflow.

Do not silently correct a recurring bad output pattern. Fix it in the prompt.
