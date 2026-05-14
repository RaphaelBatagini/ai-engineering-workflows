# Async communication workflow

> **Category:** communication
> **Typical duration:** 5–15 minutes
> **Tools:** Claude Code or Claude.ai
> **Related prompt:** [`prompts/communication/async-communication.md`](../../prompts/communication/async-communication.md)

---

## Context

This workflow structures AI-assisted drafting of asynchronous engineering communication: incident stakeholder updates, sprint status summaries, technical decision announcements, blocker escalations, handoff notes, and cross-team coordination messages.

AI produces a structured first draft from a brief the engineer writes. The engineer validates accuracy and tone before sending. The key constraint: the brief must contain verified facts — AI cannot generate accurate communication from unverified or incomplete input.

---

## When to use this workflow

- Incident stakeholder updates during or after an incident
- Sprint status reports to engineering leadership or product
- Cross-team dependency notifications
- Technical decision announcements
- Blocker escalations requiring clear, action-oriented framing
- Handoff notes for async collaboration across time zones

## When not to use this workflow

- Quick Slack messages or informal team communication — write directly
- Communication where relationship context and personalization are primary — the model cannot calibrate for relationship dynamics
- Sensitive communications: performance discussions, personnel matters, any HR-adjacent communication

---

## Prerequisites

- [ ] You know the audience and their relevant context
- [ ] The facts to communicate are accurate and verified — do not draft communication about unverified claims
- [ ] Decisions communicated as made are actually made

---

## Steps

### Step 1: Write the communication brief

**AI role:** None
**Human role:** Document what needs to be communicated before requesting a draft

Write a brief that answers:

```
Audience: [who receives this — team, engineering leadership, cross-team, stakeholders]
Format: [Slack post | email | Jira comment | doc update | PR description]
Purpose: [inform | escalate | coordinate | announce | hand off]
Key facts: [bullet list of what must be communicated — verified only]
Decisions made: [what has been decided]
Decisions pending: [what is still open — must be explicit]
Blockers: [what is blocked, on whom, with what urgency]
Tone: [informational | urgent | executive summary | technical detail]
```

> **Validation gate:** The brief contains only verified facts. If you cannot populate "key facts" with accurate information, the communication is not ready to draft.

---

### Step 2: Draft with AI

**AI role:** Produce a structured first draft from the communication brief
**Human role:** Provide the brief; evaluate the draft for accuracy, completeness, and tone

Use the [async communication prompt](../../prompts/communication/async-communication.md) with the brief.

> **Validation gate:** Read the draft against the brief. Is every key fact present? Are decisions accurately described as made vs. pending? Are blockers visible and appropriately urgent?

---

### Step 3: Validate for accuracy and tone

**AI role:** None
**Human role:** Verify every factual claim; evaluate tone against the actual situation

Work through the draft:
- Every technical claim is accurate
- No pending decision is presented as made
- Blockers are clearly stated — not softened or buried
- Numbers, timelines, and scope statements are correct
- Tone matches the actual situation — AI drafts tend toward neutral professionalism; if urgency is needed, verify it's present

> **Validation gate:** You have read every sentence and verified it. You have not approved a draft because it reads smoothly.

---

### Step 4: Revise and send

**AI role:** Optional — can revise specific sections given feedback
**Human role:** Make final edits; send

Your name is on this communication. It reflects your judgment, not the model's draft.

---

## Output format

Varies by format as specified in the brief: Slack post, email, Jira comment, document update, PR description.

---

## Quality checklist

- [ ] Every factual claim is verified
- [ ] Decisions made vs. pending are clearly distinguished
- [ ] Blockers are explicit and appropriately urgent — not buried or softened
- [ ] Tone matches the audience and the actual situation
- [ ] No AI-introduced optimism bias — the communication reflects reality

---

## Known limitations

- **Tone calibration:** AI drafts default to professional neutrality. For urgent communications, verify that urgency is conveyed accurately. The model rarely overstates urgency — review for understated urgency.
- **Sensitive context:** Do not use this workflow for personnel, performance, or other sensitive communications.
- **Relationship context:** The model cannot know the communication history, relationship dynamics, or organizational context that might affect framing. Apply this manually.

---

## Related workflows

- [Incident Investigation](../engineering/incident-investigation.md) — incident stakeholder updates specifically
- [Documentation](../delivery/documentation.md) — for more formal technical documentation
