# Guardrails

This directory contains the quality gates, review checklists, and validation criteria that apply to AI-assisted outputs across all workflows in this repository.

---

## Purpose

Guardrails are the structural mechanism that makes AI-assisted engineering reliable. They define what "acceptable" means for each type of AI-assisted output, so that review is consistent across engineers, sessions, and time.

Without explicit guardrails, review quality depends entirely on the individual engineer's attention and mood on a given day. With guardrails, the standard is defined and the review is checkable.

---

## Contents

| File | Purpose |
|---|---|
| [quality-gates.md](quality-gates.md) | Minimum quality criteria by output type: code, analysis, documentation, investigation outputs, communication, tickets |
| [review-checklist.md](review-checklist.md) | Rapid cross-cutting checklist for reviewing any AI-assisted output before use |

---

## How guardrails relate to workflows

Each workflow in [`workflows/`](../workflows/) references the relevant quality gates in its quality checklist section. The files in this directory are the canonical definitions — workflow quality checklists are derived from them, not the other way around.

When a new output type is introduced (via a new workflow), the corresponding quality gate criteria should be added to [quality-gates.md](quality-gates.md) before or alongside the workflow.

---

## Updating guardrails

Guardrails should be updated when:

- A new class of failure mode is observed in practice that existing criteria don't catch
- A criterion is consistently misinterpreted and needs clarification
- A new output type is introduced by a new workflow

Changes to guardrails affect all workflows that reference them. Review changes carefully and notify workflow maintainers when criteria change.
