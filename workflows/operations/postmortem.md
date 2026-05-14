# Postmortem workflow

> **Category:** operations
> **Typical duration:** 45–90 minutes
> **Tools:** Claude Code or Claude.ai; incident channel history; DataDog or equivalent
> **Related prompt:** [`prompts/operations/postmortem.md`](../../prompts/operations/postmortem.md)

---

## Context

This workflow structures AI-assisted postmortem documentation after a production incident. AI accelerates timeline reconstruction and document drafting from raw incident artifacts. Engineers validate accuracy, author the causal chain, and own all action items.

The goal of a postmortem is shared understanding of what happened and a concrete improvement plan — not attribution. Blameless language throughout is a requirement, not a preference.

---

## When to use this workflow

- Any production incident that affected users or data and is now resolved
- Incidents that revealed a systemic risk worth documenting and acting on
- Near-misses worth capturing before the context fades

## When not to use this workflow

- Minor operational issues with no user impact and no systemic learning — a brief Slack note is sufficient
- Active incident investigation — resolve first, document after

---

## Prerequisites

- [ ] Incident is resolved or in a stable mitigated state
- [ ] Incident channel history is accessible
- [ ] Alert timestamps and deployment records are accessible
- [ ] Participants are available to review the draft before it is finalized

---

## Steps

### Step 1: Gather raw incident artifacts

**AI role:** None
**Human role:** Collect all raw materials before beginning reconstruction

Collect:
- Full incident channel export (or screenshot history)
- Alert timestamps from DataDog or PagerDuty
- Deployment history for the affected service in the 48 hours before the incident
- Config change history if relevant
- Metrics screenshots covering the incident window
- Any communication sent to stakeholders during the incident

> **Validation gate:** Raw artifacts are assembled. Do not begin timeline reconstruction from memory alone — source artifacts reduce errors and ensure the postmortem can be validated.

---

### Step 2: Reconstruct the timeline

**AI role:** Generate a chronological timeline from the incident channel history, alert timestamps, and deployment records
**Human role:** Validate the timeline against participant memory and missing data

Provide the model with the raw artifacts. Request a chronological timeline of events from first alert to resolution.

Review the output with incident participants:
- Are missing events captured?
- Are timestamps accurate?
- Are causal links stated explicitly or just implied by sequence?
- Are there any inaccuracies introduced by the model?

Correction priority: sequence and causality are more important than precise minute-level timing.

> **Validation gate:** At least one incident participant has reviewed the timeline and confirmed its accuracy. A timeline with errors propagates those errors into the root cause analysis.

---

### Step 3: Document the root cause

**AI role:** Assist with structuring the root cause narrative from confirmed investigation findings
**Human role:** Author the causal chain; validate every claim against evidence

Root cause documentation requires three layers:

1. **Immediate trigger:** The specific event that initiated the failure (a deployment, a traffic spike, a failed dependency, a configuration change)
2. **Contributing factors:** Why the system was vulnerable to this trigger (missing circuit breaker, insufficient capacity margin, inadequate test coverage, monitoring gap)
3. **Detection gap:** Why the issue wasn't caught before it reached production, and why detection took as long as it did

Use AI to help structure the narrative. Review every causal claim against the evidence in the timeline. The model will produce plausible-sounding causal chains — validate them against actual evidence, not internal consistency.

> **Validation gate:** Every causal claim can be traced to specific evidence. "The deployment caused the outage" requires: what in the deployment, why it produced the failure mode, and what evidence confirms this.

---

### Step 4: Draft the postmortem document

**AI role:** Produce the structured postmortem document from the timeline, root cause, and impact data
**Human role:** Review every section; author all action items

Use the [postmortem prompt](../../prompts/operations/postmortem.md) to draft the document.

Review the draft for:
- **Accuracy:** Every claim verifiable against evidence
- **Blameless language:** The document names systems, processes, and conditions — not individuals
- **Impact precision:** User impact and data impact are accurate, not inflated or minimized
- **Action item quality:** Actions address root causes and contributing factors, not just symptoms

> **Validation gate:** At least one engineer not involved in the initial draft has reviewed the complete document.

---

### Step 5: Define and assign action items

**AI role:** None
**Human role:** Define concrete action items with owners and timelines

Action items must:
- Address the root cause or a contributing factor — not just add monitoring for the same failure mode
- Be specific and deliverable (a ticket, a PR, a design review)
- Have an owner and a target date
- Be tracked in the team's backlog or issue tracker

An action item without an owner and a timeline is not an action item — it is a note.

> **Validation gate:** Every action item is created in the team's issue tracker before the postmortem is finalized.

---

### Step 6: Review and publish

**AI role:** None
**Human role:** Final review; publish to the appropriate location

Final review checklist before publishing:
- Timeline is complete and accurate
- Root cause is specific and evidence-grounded
- Blameless language throughout
- Action items are created and linked
- Distribution is appropriate for the incident severity

---

## Output format

A postmortem document with: incident summary, timeline, impact assessment, root cause (trigger + contributing factors + detection gap), action items with owners and timelines.

---

## Quality checklist

- [ ] Timeline is complete — no unaccounted gaps
- [ ] Root cause is specific: named component, failure mode, triggering condition
- [ ] Impact scope is accurate — not estimated or assumed
- [ ] Contributing factors are identified, not just the immediate trigger
- [ ] Detection gap is addressed — why wasn't this caught sooner?
- [ ] Every action item has an owner, a target date, and a linked ticket
- [ ] Blameless language throughout — systems and processes, not individuals

---

## Known limitations

- **Channel history gaps:** If incident communication happened outside the main incident channel (DMs, video calls), the timeline reconstruction may be incomplete. Fill gaps from participant memory.
- **Attribution accuracy:** The model may attribute causality more confidently than the evidence supports. Review causal language carefully — "caused by" requires strong evidence; "correlated with" or "consistent with" should be used where causality is inferred rather than confirmed.

---

## Related workflows

- [Incident Investigation](../engineering/incident-investigation.md) — the investigation workflow that precedes the postmortem
- [DataDog Investigation](datadog-investigation.md) — for detailed observability analysis that feeds into the postmortem
- [Async Communication](../communication/async-communication.md) — for distributing the postmortem summary to stakeholders
