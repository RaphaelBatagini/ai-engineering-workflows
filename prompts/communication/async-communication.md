# Async communication prompt

> **Category:** communication
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/communication/async-communication.md`](../../workflows/communication/async-communication.md)

---

## Purpose

Produces a structured first draft of an async engineering communication (Slack post, email, status update, escalation) from a brief that the engineer provides. The engineer validates accuracy and tone before sending.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `brief` | Required | Structured communication brief: audience, format, purpose, key facts, decisions made, decisions pending, blockers, tone. See workflow Step 1 for the brief format. |

The brief must contain only verified facts. Do not run this prompt with unverified information.

---

## Prompt

```
You are drafting an async engineering communication. Produce a first draft from the brief below. The engineer will validate every factual claim before sending.

Communication brief:
[PLACEHOLDER: paste the full communication brief — audience, format, purpose, key facts, decisions made/pending, blockers, tone]

---

Produce a draft in the format specified in the brief.

Requirements:
- Every fact in the brief must appear in the draft
- Decisions made and decisions pending must be clearly distinguished — do not present pending decisions as made
- Blockers must be explicit and visible — do not bury them in qualifications or softening language
- Tone must match the specified audience and purpose — if tone is "urgent," convey urgency directly
- Do not introduce information not present in the brief
- Do not introduce optimism, positive framing, or reassurance not present in the brief

If the brief is missing information that is needed to produce a complete communication, note the gaps explicitly rather than filling them with assumptions.
```

---

## Output contract

A single draft in the format specified in the brief (Slack post, email, Jira comment, etc.) with:
- All key facts from the brief present
- Decisions made vs. pending clearly distinguished
- Blockers visible and appropriately urgent
- Tone matching the specified audience

---

## Validation criteria

- [ ] Every fact in the brief is present in the draft
- [ ] No information was added that wasn't in the brief
- [ ] Decisions made vs. pending are correctly distinguished
- [ ] Blockers are visible — not buried or softened
- [ ] Tone matches the specified audience and urgency level

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Optimism bias | Draft is more positive or reassuring than the brief describes | Add to prompt: "Do not introduce positive framing not present in the brief" |
| Missing blockers | Blockers from the brief are absent or buried in the draft | Add to prompt: "Blockers must be clearly stated and visible, not embedded in qualifications" |
| Pending as decided | The draft presents a pending decision as made | Add to brief: distinguish "decided" from "under discussion" explicitly |
| Tone mismatch | Urgent communication reads as informational, or vice versa | Be more explicit about tone in the brief: "tone: URGENT — action required by EOD" |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
