# Workflows

This directory contains process specifications for AI-assisted engineering tasks. Each workflow defines the steps, the AI role, the human role, and the validation gates for a specific engineering task.

---

## What a workflow specification is

A workflow specification is a process document — not a tutorial and not a prompt. It answers:

- When should I use this workflow?
- What do I need before starting?
- What does the AI contribute at each step?
- What do I contribute and validate at each step?
- What must be true before I proceed to the next step?
- What does a complete, acceptable output look like?

Workflows are designed to be followed, not read once and then improvised from. The validation gates at each step are mandatory — they are the structural mechanism that distinguishes this from a prompt collection.

---

## Directory structure

```
workflows/
├── engineering/     # PR review, code review, debugging, architecture review, incidents, decisions
├── delivery/        # Ticket refinement, documentation, onboarding
├── communication/   # Async updates, status reports
└── operations/      # DataDog investigation, postmortems
```

---

## Workflow selection guide

| I need to... | Workflow |
|---|---|
| Review a pull request with AI assistance | [PR Review](engineering/pr-review.md) |
| Conduct a systematic code quality review | [Code Review](engineering/code-review.md) |
| Debug a problem with structured hypothesis generation | [Debugging](engineering/debugging.md) |
| Review an architecture proposal | [Architecture Review](engineering/architecture-review.md) |
| Investigate a production incident | [Incident Investigation](engineering/incident-investigation.md) |
| Make and document a technical decision | [Technical Decision](engineering/technical-decision.md) |
| Refine a ticket or user story | [Ticket Refinement](delivery/ticket-refinement.md) |
| Generate or improve engineering documentation | [Documentation](delivery/documentation.md) |
| Prepare onboarding materials | [Onboarding](delivery/onboarding.md) |
| Draft an async update or status report | [Async Communication](communication/async-communication.md) |
| Investigate a DataDog alert or anomaly | [DataDog Investigation](operations/datadog-investigation.md) |
| Write a postmortem | [Postmortem](operations/postmortem.md) |

---

## Workflow conventions

Every workflow follows the same structure. See [templates/workflow-template.md](../templates/workflow-template.md) for the full template.

Key conventions:
- **AI role / Human role** are explicitly defined at each step
- **Validation gates** are mandatory — they are checkpoints, not suggestions
- **Quality checklists** are at the end of every workflow for pre-acceptance review
- **Known limitations** are documented — workflows are honest about where they don't work well

---

## Adding a new workflow

See [CONTRIBUTING.md](../CONTRIBUTING.md) for authoring requirements and the review process.
Use the [workflow template](../templates/workflow-template.md) as your starting structure.
All new workflows require peer review before merge.
