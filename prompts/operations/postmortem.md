# Postmortem prompt

> **Category:** operations
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/operations/postmortem.md`](../../workflows/operations/postmortem.md)

---

## Purpose

Produces a structured postmortem draft from incident artifacts — timeline notes, alert data, log excerpts, and remediation actions. The output provides a starting structure that incident participants validate, correct, and complete with the contextual judgment that only people present during the incident can provide.

AI drafts the structure; engineers own the accuracy and the narrative.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `incident_summary` | Required | Brief description of the incident: what failed, when, and the observed impact (user-facing and internal). |
| `timeline_notes` | Required | Raw timeline notes from the incident. Timestamps, observations, and actions taken. Does not need to be structured — raw Slack threads, PagerDuty notes, or engineer notes are appropriate. |
| `root_cause_notes` | Required | Initial root cause observations from the post-incident investigation. Can be preliminary — the model will help structure them into the 3-layer format. |
| `remediation_actions` | Required | Actions already taken (immediate mitigations, permanent fixes deployed) and actions identified but not yet complete. |
| `impact_data` | Optional | Quantified impact: duration, affected users, error rate, revenue or SLA impact. Include if available — without it, the impact section will contain placeholders. |
| `detection_notes` | Optional | How the incident was detected (alert, customer report, engineer observation) and any notes on detection gaps. |

---

## Prompt

```
You are drafting a postmortem from incident artifacts. Your role is to structure the provided information into a clear, blameless postmortem. The incident participants will validate the timeline, correct the root cause analysis, complete missing sections, and approve before publication.

Use blameless language throughout. The postmortem documents what happened in the system — not what any individual did wrong.

**Incident summary:**
[incident_summary]

**Timeline notes (raw):**
[timeline_notes]

**Root cause notes:**
[root_cause_notes]

**Remediation actions:**
[remediation_actions]

**Impact data:**
[impact_data — or "Not provided; insert placeholder"]

**Detection notes:**
[detection_notes — or "Not provided; insert placeholder"]

---

Generate a postmortem draft with the following sections:

**1. Incident summary**
One paragraph: what failed, when it started and ended, and the user-facing impact. No root cause in this section.

**2. Impact**
Structured impact summary: duration, affected services, user impact, and any quantified metrics (error rate, affected users, revenue impact). Use provided impact data; insert [NEEDS DATA: ...] where data was not provided.

**3. Timeline**
Chronological list of events from the timeline notes. Format: `HH:MM UTC — [event description]`. Reconstruct the timeline from the provided notes; do not fabricate events. Flag gaps:
> [TIMELINE GAP: [timeframe] — no notes for this period; review with incident participants]

**4. Root cause analysis**
Structure the root cause using three layers:
- **Immediate trigger:** The direct technical cause — the specific failure that cascaded into the incident
- **Contributing factors:** System conditions that made the failure possible or made it worse than it otherwise would have been
- **Detection gap:** Why the incident wasn't caught sooner — what monitoring, alerting, or runbook gaps allowed the failure to propagate

**5. What went well**
Elements that worked as intended: detection mechanisms that fired, runbook steps that were followed correctly, communication that was effective. Derive from timeline notes; do not fabricate.

**6. What needs improvement**
Gaps identified by the incident: in monitoring, alerting, runbooks, system resilience, or process. Derive from root cause analysis and timeline gaps.

**7. Action items**
Structured action items derived from the remediation actions and improvement gaps. Format as a table:

| Action | Owner | Due date | Status |
|---|---|---|---|
| [action] | [ASSIGN] | [SET] | Open |

Insert [ASSIGN] and [SET] placeholders — owners and due dates must be assigned by the team, not the model.

Use blameless language throughout. Do not assign personal blame. Do not include meta-commentary.
```

---

## Output contract

The model should return a complete postmortem draft with all seven sections. The draft should:
- Use blameless language — no individual blame, no "engineer X forgot to..."
- Structure root cause using the three-layer format (trigger / contributing factors / detection gap)
- Include `[TIMELINE GAP: ...]` flags for periods not covered by timeline notes
- Include `[NEEDS DATA: ...]` placeholders where quantified impact data was not provided
- Include `[ASSIGN]` and `[SET]` placeholders in action items — never assign owners or due dates
- Contain no meta-commentary

---

## Validation criteria

- [ ] Timeline accurately reflects the provided notes — no fabricated events
- [ ] Timeline gaps are explicitly flagged, not smoothed over with guesses
- [ ] Root cause uses the 3-layer structure (trigger / contributing factors / detection gap)
- [ ] Language is blameless — no statements attribute failure to individual error
- [ ] Action items are concrete and testable, not vague ("improve monitoring" is too vague; "add alert on [metric] threshold [X]" is acceptable)
- [ ] "What went well" is grounded in the timeline, not generic praise
- [ ] Action items have `[ASSIGN]` and `[SET]` placeholders filled in by the team, not the model

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Blame language | Output uses phrases like "the engineer forgot", "human error", "X failed to..." | Reject affected sections; re-run with: "Use only blameless language. Describe system behavior and process gaps, not individual actions." |
| Fabricated timeline events | Timeline includes events not present in provided notes | Cross-reference every timeline entry against input notes; reject fabricated events and re-run with: "Only include events from the provided notes. Use [TIMELINE GAP] for missing periods." |
| Shallow root cause | Root cause section collapses to a single sentence or describes only the immediate trigger without contributing factors or detection gap | Re-run with: "The root cause analysis must address all three layers: immediate trigger, contributing factors, and detection gap. For each layer, be specific." |
| Generic action items | Action items like "improve alerting" or "write runbook" without specifics | Reject; re-run with: "Each action item must be specific and testable. What alert? On what metric? For what runbook step?" |
| Missing impact data handled incorrectly | Model estimates or invents impact numbers instead of inserting placeholders | All estimated or unavailable impact data must use [NEEDS DATA] placeholders. Re-run with explicit instruction. |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
