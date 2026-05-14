# Getting started

This guide covers how to use the workflows, prompts, and standards in this repository, and how to contribute new content.

---

## What this repository is

A structured library of AI-assisted engineering workflows for experienced engineers. Each workflow defines the steps, the AI role, the human role, and the validation gates for a specific engineering task.

If you're new to this repository, start with [PHILOSOPHY.md](../../PHILOSOPHY.md) for the engineering principles, then return here.

---

## How to use a workflow

1. **Find the workflow** for your task in [`workflows/`](../../workflows/) or the [workflow index in the README](../../README.md#workflow-index)
2. **Read the full workflow** before starting — understand the prerequisites, steps, and validation gates
3. **Satisfy the prerequisites** — workflows require specific inputs to be prepared before the first step. Do not skip this
4. **Follow the steps** — each step defines the AI role, the human role, and a validation gate. Validation gates are mandatory
5. **Apply the quality checklist** before accepting the final output

If a workflow's prerequisites aren't met, stop and gather the required context before proceeding. Workflows run with incomplete context produce low-quality outputs that require more revision time than the workflow saved.

---

## How to use a prompt

1. **Find the prompt** corresponding to the workflow step you're on in [`prompts/`](../../prompts/)
2. **Read the input contract** — collect every required input before filling in the prompt
3. **Fill in placeholders** with the actual context for your session
4. **Run the prompt** in Claude Code, Claude.ai, or your configured MCP-enabled tool
5. **Evaluate the output** against the validation criteria in the prompt file
6. **Check failure modes** if output quality is low — the failure modes section describes common bad outputs and how to address them

---

## Tools this repository references

**Claude Code:** Anthropic's AI coding tool. The primary tool referenced in workflow examples.
[docs.anthropic.com/en/docs/claude-code](https://docs.anthropic.com/en/docs/claude-code)

**MCP (Model Context Protocol):** Protocol enabling AI tools to connect to external systems (GitHub, Jira, DataDog). Required for the MCP-enabled workflow examples.
[modelcontextprotocol.io](https://modelcontextprotocol.io)

**GitHub, Jira, DataDog:** The specific integrations referenced in workflow examples. The workflow logic is tool-informed but not tool-dependent — adapt for your team's toolchain.

---

## Understanding the validation gates

Every workflow step includes a validation gate — a specific condition that must be true before proceeding. These are not optional review suggestions. They are the structural mechanism that ensures AI output is validated before it propagates into subsequent steps.

When a validation gate fails:
- Do not proceed to the next step
- Address the failure: revise the prompt, gather missing context, or produce the output manually
- If a validation gate consistently fails, the prompt or workflow likely needs updating

---

## Review standards

Before accepting any AI-assisted output, apply:
- The [review checklist](../../guardrails/review-checklist.md) — applies to all output types
- The relevant [quality gates](../../guardrails/quality-gates.md) — per output type (code, analysis, documentation, etc.)

The fundamental question before accepting any output: *"Can I stand behind this as my own professional judgment?"*

---

## Contributing

See [CONTRIBUTING.md](../../CONTRIBUTING.md) for:
- How to add new workflows and prompts
- Naming conventions and template requirements
- The review process for new contributions
- What not to contribute

---

## Repository structure reference

```
ai-engineering-workflows/
├── README.md               ← Start here
├── PHILOSOPHY.md           ← Engineering principles
├── CONTRIBUTING.md         ← How to contribute
├── workflows/              ← Process specifications
├── prompts/                ← Prompt specifications
├── guardrails/             ← Quality gates and review checklists
├── docs/standards/         ← Authoring standards
├── examples/               ← Representative session artifacts
├── templates/              ← Scaffolding for new contributions
└── docs/onboarding/        ← This file
```
