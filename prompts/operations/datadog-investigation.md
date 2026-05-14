# DataDog investigation prompt

> **Category:** operations
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/operations/datadog-investigation.md`](../../workflows/operations/datadog-investigation.md)

---

## Purpose

Analyzes raw DataDog observability data to identify signal patterns, surface candidate root causes, and suggest targeted follow-up queries for confirmation or elimination.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `alert_context` | Required | Alert name, triggered condition (metric + threshold + duration), alert time window. |
| `metrics_data` | Required | Error rates, latency, and relevant metrics for the affected service in the alert window. Include the pre-alert baseline for comparison if available. |
| `log_excerpts` | Required* | Relevant log lines from the alert window with timestamps. *Required if logs are available — note sampling rate if known. |
| `service_topology` | Optional | Brief description of affected service dependencies: upstream producers, downstream consumers, shared infrastructure. Significantly improves correlation analysis. |
| `deployment_context` | Optional | Deployments and config changes in the 48 hours before the alert. Often the most immediately useful context. |

---

## Prompt

```
You are assisting with a DataDog alert investigation. Analyze the provided observability data and produce a structured investigation output.

Service topology (if provided):
[PLACEHOLDER: paste service_topology, or omit]

Deployment context (if provided):
[PLACEHOLDER: paste deployment_context — recent deployments and config changes, or omit]

Alert context:
[PLACEHOLDER: paste alert name, triggered condition, and time window]

Metrics data (error rates, latency, relevant metrics with timestamps):
[PLACEHOLDER: paste metrics data — include the pre-alert baseline if available]

Log excerpts (with timestamps):
[PLACEHOLDER: paste relevant log lines, or note "not available"]

---

Produce the following:

**1. Signal summary**
What does the observability data show? Describe the pattern of degradation: onset, progression, affected components, correlation between signals.

**2. Candidate root causes**
Ranked list of root cause candidates consistent with the observed signals. For each:
- **Candidate:** Specific description of the failure mode
- **Supporting signals:** Specific data points that support this candidate
- **Against this candidate:** What in the data is inconsistent with this explanation
- **Confirmation query:** A specific DataDog query or action that would confirm this candidate

**3. Correlation analysis**
Which signals appear causally related vs. coincidental? Note any signals that appear anomalous relative to the pattern.

**4. Suggested next queries**
3–5 specific DataDog queries (log searches, metric queries, or trace searches) to narrow the investigation. Include the query purpose for each.

Qualify all interpretations appropriately. If the available data is insufficient to rank candidates confidently, say so and describe what additional data would help.
```

---

## Output contract

1. **Signal summary:** Pattern description with onset, progression, and affected components
2. **Candidate root causes:** Ranked list with evidence assessment and confirmation queries per candidate
3. **Correlation analysis:** Causal vs. coincidental signal assessment
4. **Suggested next queries:** Specific queries with stated purpose

---

## Validation criteria

- [ ] Signal summary describes the specific pattern, not just "an increase in errors"
- [ ] Candidates are specific failure modes — not "database issue" but "connection pool exhaustion under concurrent request load"
- [ ] Confirmation queries are specific and executable in DataDog
- [ ] Correlation analysis distinguishes causation from coincidence

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Generic candidates | "Database issue," "network latency," "memory pressure" without specifics | Add more specific metrics data and log excerpts |
| Missing correlation analysis | Each signal analyzed independently, no cross-service correlation | Add `service_topology`; append: "Explicitly analyze which signals appear causally related" |
| Inaccurate DataDog syntax | Suggested queries use invalid DataDog syntax | Validate queries against DataDog documentation before executing |
| Overconfident root cause | Single candidate presented without alternatives | Append: "Provide at least 3 candidates, not just the most likely one" |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
