# Incident investigation prompt

> **Category:** engineering
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/engineering/incident-investigation.md`](../../workflows/engineering/incident-investigation.md)

---

## Purpose

Generates ranked hypotheses for an active or post-incident investigation from a symptom timeline and observability artifacts. Also used to structure root cause narrative for postmortem documentation.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `incident_description` | Required | Documented scope and timeline anchor: what is affected, when it started, initial impact estimate, recent changes. |
| `observability_data` | Required | Raw observability artifacts: error rate snapshots, log excerpts with timestamps, latency data, relevant traces. Not summaries — raw data. |
| `service_context` | Optional | Brief description of affected service(s): purpose, architecture, key dependencies, known failure modes or technical debt. |
| `investigation_mode` | Required | One of: `active` (generating hypotheses during investigation) or `post-incident` (structuring root cause narrative for postmortem). |

---

## Prompt (active investigation mode)

```
You are assisting with an active incident investigation. Generate ranked hypotheses for the root cause based on the provided timeline and observability data.

Investigation mode: active

Service context (if provided):
[PLACEHOLDER: paste service_context, or omit]

Incident description (scope, timeline anchor, recent changes):
[PLACEHOLDER: paste the incident description]

Observability data (error rates, logs, latency, traces):
[PLACEHOLDER: paste the raw observability artifacts]

---

For each hypothesis, provide:
- **Hypothesis:** Specific causal statement — not "a dependency issue" but "the database connection pool is exhausted because [specific condition] under [specific trigger]"
- **Supporting signals:** Specific data points from the provided observability data that support this
- **Contradicting signals:** What in the data argues against this
- **Confirmation query:** The specific DataDog query, log search, or action that would confirm this
- **Priority:** High / Medium / Low based on consistency with available evidence and potential blast radius

List 3–5 hypotheses. Do not include hypotheses not grounded in the provided data.

After the hypothesis list, note any investigation signals that appear anomalous or unexplained by the current hypothesis set.
```

---

## Prompt (post-incident mode)

```
You are assisting with post-incident root cause analysis. Structure a root cause narrative from the provided incident data.

Investigation mode: post-incident

Service context (if provided):
[PLACEHOLDER: paste service_context, or omit]

Incident timeline and resolution summary:
[PLACEHOLDER: paste the confirmed timeline and resolution]

Observability data from the incident window:
[PLACEHOLDER: paste the relevant observability artifacts]

---

Structure the root cause analysis with three layers:

**1. Immediate trigger:** The specific event that initiated the failure. What happened, when, and in which component.

**2. Contributing factors:** Why the system was vulnerable to this trigger. What conditions were in place that turned this trigger into an incident rather than a handled failure.

**3. Detection gap:** Why the issue wasn't caught before it reached users. What would need to be true for this to be detected earlier (in testing, monitoring, or alerting).

For each claim: cite the specific evidence that supports it. If a claim cannot be grounded in the provided evidence, qualify it as a hypothesis rather than a confirmed cause.
```

---

## Output contract

**Active mode:**
1. Hypothesis list (3–5) with supporting/contradicting signals, confirmation queries, and priority
2. Anomalous signals not explained by the current hypothesis set

**Post-incident mode:**
1. Immediate trigger with evidence citation
2. Contributing factors with evidence citations
3. Detection gap analysis

---

## Validation criteria

- [ ] Every hypothesis is grounded in specific evidence from the provided data
- [ ] Confirmation queries are specific and executable
- [ ] Post-incident root cause claims are supported by citations — not plausible-sounding narratives
- [ ] Detection gap addresses why monitoring or alerting didn't catch this earlier

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Vague hypotheses | "Network issue," "database problem" without specifics | Provide more specific observability data — error messages, specific metrics |
| Overconfident root cause | "The root cause is X" without evidence qualification | Append: "Qualify all causal claims with the evidence that supports them" |
| Missing detection gap | Post-incident output only covers the trigger, not why it wasn't caught | Explicitly ask for detection gap analysis in the prompt |
| Hypothesis not grounded in data | Hypothesis appears plausible but cites no evidence from the provided data | Add: "Only include hypotheses directly supported by the provided observability data" |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
