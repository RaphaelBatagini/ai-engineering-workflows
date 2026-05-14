# Prompts

This directory contains prompt specifications for AI-assisted engineering tasks, organized by workflow category.

---

## What a prompt specification is

A prompt specification is not just the text you send to a model. It is a complete engineering artifact that includes:

- **Input contract** — what context the engineer must provide before running the prompt
- **Prompt body** — the instructions sent to the model, with placeholders for variable inputs
- **Output contract** — the expected structure and format of the model's response
- **Validation criteria** — how the engineer evaluates whether the output is acceptable
- **Failure modes** — known bad outputs, how to identify them, and corrective actions

This structure makes prompts reviewable, maintainable, and consistent across engineers and sessions.

---

## Directory structure

```
prompts/
├── engineering/     # PR review, code review, debugging, architecture review, incident investigation
├── delivery/        # Ticket refinement, documentation, onboarding
├── communication/   # Async updates, status reports
└── operations/      # DataDog investigation, postmortem
```

Each prompt file corresponds to a step or workflow in [`workflows/`](../workflows/). Files are linked bidirectionally — prompt files link to their workflow and workflows link to their prompt files.

---

## How to use a prompt

1. Open the prompt file for the relevant task
2. Read the **input contract** — collect all required context before running the prompt
3. Fill in the **placeholders** in the prompt body with the actual session context
4. Run the prompt in your AI tool (Claude Code, Claude.ai, or via MCP integration)
5. Evaluate the output against the **validation criteria**
6. If output quality is low, consult the **failure modes** section before revising

Do not skip the input contract step. Running a prompt without the required context is the most common cause of low-quality outputs and unnecessary revision cycles.

---

## Relationship between prompts and workflows

Workflows (in [`workflows/`](../workflows/)) define the **process** — the sequence of steps, the human role at each step, and the validation gates. Prompts define the **AI input layer** — what you ask the model at specific steps.

A workflow may reference one or more prompts. A prompt is always used in the context of a workflow step — not as a standalone interaction without process context.

---

## Adding a new prompt

See [CONTRIBUTING.md](../CONTRIBUTING.md) for authoring requirements and the review process.
Use the [prompt template](../templates/prompt-template.md) as your starting structure.
All new prompts require peer review before merge.

For authoring guidance, see [docs/standards/prompt-standards.md](../docs/standards/prompt-standards.md).
