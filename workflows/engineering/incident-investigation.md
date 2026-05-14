# Incident investigation workflow

> **Category:** engineering
> **Typical duration:** Variable — structured for active incidents and post-incident analysis
> **Tools:** Claude Code or Claude.ai; DataDog or equivalent observability; incident channel/runbook
> **Related prompt:** [`prompts/engineering/incident-investigation.md`](../../prompts/engineering/incident-investigation.md)

---

## Context

This workflow structures AI assistance during and after incident investigations. It operates in two modes:

- **Active investigation:** AI accelerates timeline reconstruction, hypothesis generation, and impact scoping during a live incident
- **Post-incident analysis:** AI accelerates postmortem documentation and root cause write-up after resolution

The engineer leads the investigation throughout. AI compresses the time spent on information synthesis and hypothesis generation. Engineering judgment drives all conclusions, escalation decisions, and external communication.

---

## When to use this workflow

- Active production incident: service degradation, outage, data integrity issue, security event
- Post-incident: writing a postmortem or root cause analysis
- Preparing an incident summary for stakeholders

## When not to use this workflow

- Routine debugging not affecting production users — use [Debugging](debugging.md)
- Alerts with a known runbook that covers the resolution path — follow the runbook directly

---

## Prerequisites

**Active investigation:**
- [ ] Incident is declared and an incident channel or bridge is open
- [ ] You have access to observability data (logs, metrics, traces)
- [ ] Initial severity and scope are estimated

**Post-incident analysis:**
- [ ] Incident is resolved or mitigated
- [ ] A timeline is reconstructable from channel history, alert timestamps, and deployment records
- [ ] The immediate fix or mitigation is documented

---

## Steps: active investigation

### Step 1: Establish timeline and scope snapshot

**AI role:** None
**Human role:** Document initial signals immediately

Document and pin to the incident channel:

```
Incident start: [timestamp when issue first observed or alerted]
Impact: [what is degraded or unavailable]
Scope: [which users, regions, or services affected — initial estimate]
Recent changes: [deployments or config changes in the last 24h]
Alert context: [alert name, threshold, triggered value]
```

Do this before anything else. The timeline anchor and initial scope statement are the foundation of the investigation. They also prevent scope confusion if additional engineers join.

> **Validation gate:** Timeline anchor and initial impact scope are pinned in the incident channel before investigation begins.

---

### Step 2: Gather observability data

**AI role:** Can suggest relevant queries based on the alert context
**Human role:** Execute queries; collect and validate raw data

Collect from your observability stack:
- Error rates and latency for affected services in the incident window
- Log excerpts from the failure window with timestamps
- Relevant traces if available
- Infrastructure metrics if relevant
- Upstream and downstream service metrics (propagation analysis)

For DataDog-specific investigation steps, see [DataDog Investigation](../operations/datadog-investigation.md).

> **Validation gate:** You have raw observability artifacts — not summaries. Provide raw data to the model for analysis, not your interpretation of it.

---

### Step 3: Generate and triage hypotheses

**AI role:** Generate ranked hypotheses from the symptom pattern, timeline, and observability data
**Human role:** Provide assembled context; triage output against system knowledge; direct investigation

Use the [incident investigation prompt](../../prompts/engineering/incident-investigation.md) with the timeline, scope, and observability data.

Triage the output immediately:
- Eliminate hypotheses incompatible with known system facts
- Prioritize by likelihood and blast radius — high-impact root causes investigated first
- Add hypotheses the model didn't generate based on your system knowledge

> **Validation gate:** Investigation is focused on 1–2 hypotheses. Unfocused parallel investigation during an active incident extends duration. Triage is mandatory.

---

### Step 4: Investigate and narrow

**AI role:** Assist with log analysis, code path tracing, query construction
**Human role:** Execute targeted investigation; update the incident channel continuously

Work through prioritized hypotheses:
- Post findings to the incident channel as you go — not at the end
- Eliminate hypotheses explicitly when ruled out — do not silently drop them
- Revise the impact scope as you learn more (it may be larger or smaller than the initial estimate)

> **Validation gate:** Before concluding a hypothesis is confirmed, you have direct evidence. Symptoms consistent with a hypothesis are not confirmation.

---

### Step 5: Mitigate and confirm resolution

**AI role:** Can assist with drafting stakeholder update text; can assist with mitigation step documentation
**Human role:** Execute mitigation; author all external communication; verify resolution

Execute the mitigation. Document in the channel:
- What the mitigation is (exactly what was done)
- When it was applied
- What changed in system behavior after (metrics confirming improvement)

> **Validation gate:** Resolution is confirmed by metric recovery — not by the mitigation action alone. Do not declare the incident resolved until observability data shows recovery.

---

## Steps: post-incident analysis

### Step 6: Reconstruct and validate the timeline

**AI role:** Reconstruct a chronological timeline from incident channel history, alert timestamps, and deployment records
**Human role:** Validate the timeline for accuracy; fill gaps from memory or additional sources

Provide the model with the incident channel export, alert timestamps, and deployment history.

Review the output for:
- Missing events you know happened but aren't captured in the raw data
- Timestamp inconsistencies
- Causal relationships implied but not stated

> **Validation gate:** Incident participants have reviewed the timeline before it becomes the foundation of the postmortem. A timeline with errors propagates those errors into the root cause analysis.

---

### Step 7: Structure the root cause analysis

**AI role:** Assist with structuring the root cause narrative from confirmed investigation findings
**Human role:** Confirm the root cause; author the causal chain

Root cause documentation requires three layers:
1. **Immediate trigger:** What specifically happened that initiated the failure
2. **Contributing factors:** Why the system was vulnerable to this trigger
3. **Detection gap:** Why this wasn't caught in code review, testing, or monitoring

Use AI to help structure the narrative. Review every causal claim against the evidence in the timeline — plausible-sounding causal chains are not the same as evidence-grounded ones.

> **Validation gate:** Every causal claim in the root cause analysis can be traced to specific evidence in the timeline or observability data. No claim is asserted without a citation.

---

### Step 8: Draft the postmortem

**AI role:** Draft the postmortem document from the validated timeline, root cause, and impact data
**Human role:** Review every section; author all action items; own the final document

Use the model to produce a postmortem draft. Review the draft against the [Postmortem workflow](../operations/postmortem.md).

> **Validation gate:** At least one incident participant has reviewed the complete postmortem (Level 2 validation per [validation-standards.md](../../docs/standards/validation-standards.md)). Action items have owners and timelines.

---

## Output format

**Active investigation:** Incident channel with documented timeline, root cause, mitigation record, and resolution confirmation
**Post-incident:** Postmortem document with timeline, root cause, impact, contributing factors, and action items

---

## Quality checklist

- [ ] Timeline is complete — no unaccounted gaps greater than a few minutes
- [ ] Root cause is specific: named component, failure mode, triggering condition
- [ ] Impact scope is accurate — not estimated or assumed
- [ ] Resolution is verified by metric recovery
- [ ] Action items have owners and timelines
- [ ] Blameless language throughout — the postmortem names systems and processes, not individuals

---

## Known limitations

- **Active incident speed pressure:** This workflow adds structure that takes a few minutes to establish at the start of an incident. For incidents with a known runbook resolution path, skip directly to the runbook.
- **Log volume:** High-volume log contexts may exceed model context limits. Pre-filter to the relevant time window and affected service before providing to the model.
- **Novel failure modes:** Hypothesis generation is grounded in known failure patterns. Novel or unusual failure modes may not appear in the hypothesis list. Always allow for root causes the model didn't generate.

---

## Related workflows

- [Debugging](debugging.md) — for non-incident production issues
- [DataDog Investigation](../operations/datadog-investigation.md) — for observability-specific investigation steps
- [Postmortem](../operations/postmortem.md) — for the full postmortem documentation workflow
- [Async Communication](../communication/async-communication.md) — for drafting stakeholder updates during or after the incident
