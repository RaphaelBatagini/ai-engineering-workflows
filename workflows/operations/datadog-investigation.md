# DataDog investigation workflow

> **Category:** operations
> **Typical duration:** 15–45 minutes
> **Tools:** Claude Code or Claude.ai; DataDog (direct access or via MCP)
> **Related prompt:** [`prompts/operations/datadog-investigation.md`](../../prompts/operations/datadog-investigation.md)

---

## Context

This workflow structures AI-assisted investigation of DataDog alerts, metric anomalies, and log-based signals. AI accelerates signal interpretation, query construction, and hypothesis generation from observability data. Engineers apply system topology knowledge and baseline familiarity to reach conclusions.

This workflow covers alert investigation and anomaly triage as a standalone operation. For active production incidents, use the [Incident Investigation workflow](../engineering/incident-investigation.md), which incorporates this workflow as a component.

---

## When to use this workflow

- Investigating a DataDog alert without a deterministic runbook resolution path
- Analyzing a metric anomaly of unclear origin
- Correlating signals across services to trace a performance degradation
- Building log queries for a complex or unfamiliar investigation pattern

## When not to use this workflow

- Alerts with a runbook that covers the resolution — follow the runbook
- Active incident investigation — use [Incident Investigation](../engineering/incident-investigation.md)
- Routine metrics review with no anomaly present

---

## Prerequisites

- [ ] The alert or anomaly is identified and the time window is known
- [ ] DataDog access is available (direct or via MCP)
- [ ] The affected service(s) are known or suspected

---

## Steps

### Step 1: Document the signal

**AI role:** None
**Human role:** Describe the alert or anomaly precisely

Document:
- Alert name and triggered condition (metric, threshold, duration)
- Time window: when did it start? Is it ongoing or resolved?
- Affected service(s) or infrastructure component(s)
- Initial impact estimate: user-facing? backend only? data at risk?
- Correlated alerts in the same window

> **Validation gate:** The signal is described precisely enough that someone unfamiliar with the service could query for it.

---

### Step 2: Gather raw observability data

**AI role:** Can suggest relevant DataDog queries based on the alert context
**Human role:** Execute queries; collect and validate raw data

Collect from DataDog:
- Error rate and latency for the affected service in the alert window
- Relevant log excerpts: error logs, slow query logs, timeout logs
- Infrastructure metrics if relevant: CPU, memory, network I/O, disk
- Upstream service metrics: is this a propagation from a dependency?
- Downstream service metrics: is this causing cascading degradation?
- Deployment and config change events in the alert window

For query assistance, the [DataDog investigation prompt](../../prompts/operations/datadog-investigation.md) can generate query suggestions from the alert context.

> **Validation gate:** You have raw DataDog data artifacts — not summaries or interpretations. Provide raw data to the model for analysis in Step 3.

---

### Step 3: AI-assisted signal analysis

**AI role:** Interpret the observability data; identify patterns, correlations, and anomaly signatures; generate ranked hypotheses
**Human role:** Provide the raw data; evaluate the analysis against system topology knowledge

Use the [DataDog investigation prompt](../../prompts/operations/datadog-investigation.md) with the collected observability data.

Expected output:
- Signal interpretation: what the data pattern indicates
- Correlation analysis: which signals appear causally related vs. coincidental
- Ranked candidate root causes consistent with the data
- Suggested additional queries to confirm or rule out each candidate

> **Validation gate:** Evaluate every interpretation against your knowledge of the system's normal operating ranges and topology. The model cannot know your system's baseline behavior — you must apply that context explicitly.

---

### Step 4: Focused investigation

**AI role:** Assist with targeted query construction and log pattern analysis
**Human role:** Execute queries; narrow to root cause; document findings

Based on the validated candidate list:
1. Run targeted queries to confirm or eliminate each candidate
2. Document findings explicitly: confirmed / ruled out / inconclusive — with evidence
3. Narrow to a probable root cause, or escalate if the investigation is inconclusive

> **Validation gate:** You have direct evidence for the root cause before concluding the investigation. "Consistent with the data" is not the same as "confirmed by the data."

---

### Step 5: Document findings and next steps

**AI role:** Can assist with drafting the investigation summary
**Human role:** Author the summary; determine and own next steps

Document:
- Confirmed root cause or highest-confidence hypothesis with qualifying language
- Supporting evidence (specific metrics, log lines, timestamps)
- Impact scope
- Action taken or recommended next step

If the investigation escalates to an incident, transition to the [Incident Investigation workflow](../engineering/incident-investigation.md).

> **Validation gate:** The summary clearly distinguishes confirmed facts from working hypotheses.

---

## Output format

An investigation summary with: signal description, findings, root cause (confirmed or qualified hypothesis), and recommended action.

---

## Quality checklist

- [ ] Root cause claim is qualified: confirmed (with evidence) vs. probable (with supporting signals) vs. possible (hypothesis only)
- [ ] Supporting evidence is cited — not just the conclusion
- [ ] Impact scope is bounded, not worst-case assumed without evidence
- [ ] Next steps are concrete and assigned

---

## Known limitations

- **Baseline knowledge:** The model cannot know your system's normal operating ranges. Anomaly interpretation requires the engineer's knowledge of what normal looks like for this service.
- **Topology knowledge:** Cross-service correlation analysis is significantly more effective when the engineer provides service dependency context in the prompt. Make topology explicit.
- **DataDog query syntax drift:** DataDog query syntax and available metric names evolve. Validate AI-suggested queries against current DataDog documentation before executing.
- **Log sampling:** If logs are sampled in production, the absence of a log pattern in the sample is not absence of the pattern overall. Note sampling rates when interpreting log-based evidence.

---

## Related workflows

- [Incident Investigation](../engineering/incident-investigation.md) — when the DataDog investigation escalates to an active incident
- [Postmortem](postmortem.md) — for post-incident documentation after resolution
