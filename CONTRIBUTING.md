# Contributing

This document defines how to contribute new workflows, prompts, and examples to this repository.

---

## Contribution types

| Type | Directory | Template |
|---|---|---|
| Workflow | `workflows/<category>/` | [`templates/workflow-template.md`](templates/workflow-template.md) |
| Prompt | `prompts/<category>/` | [`templates/prompt-template.md`](templates/prompt-template.md) |
| Example | `examples/<tool>/` | See [Examples](#examples) below |
| Architecture Decision Record | `docs/decisions/` | [`templates/adr-template.md`](templates/adr-template.md) |

---

## Naming conventions

### Files

- Use lowercase kebab-case: `pr-review.md`, `datadog-investigation.md`
- Be specific: `incident-investigation.md` not `incidents.md`
- Use verb-noun or noun-noun format for workflows: `ticket-refinement.md`, `architecture-review.md`

### Directories

- Category names are lowercase, single-word or hyphenated: `engineering/`, `mcp-workflows/`
- Do not create single-file directories — if only one file exists, reconsider the category

### Titles

- H1 headings: sentence case — `# PR review workflow`
- Workflow index table: title case — `PR Review`

---

## Workflow authoring standards

Every workflow file must:

1. Follow the [workflow template](templates/workflow-template.md) structure exactly
2. Define at least one explicit human validation gate per major step
3. Specify the AI role and human role distinctly for each step
4. Include a quality checklist that can be evaluated independently of the full workflow
5. List at least two known limitations
6. Link to the corresponding prompt file(s) in `prompts/`

Workflows that do not include explicit human review gates at each step will not be merged. The validation gates are not suggestions — they are the structural mechanism that distinguishes this repository from a prompt collection.

---

## Prompt authoring standards

Every prompt file must:

1. Follow the [prompt template](templates/prompt-template.md) structure exactly
2. Define an input contract — what context is required before the prompt is run
3. Define an output contract — the expected structure and format of the response
4. List validation criteria — how the engineer evaluates the output
5. List at least two failure modes with identification signals and corrective actions
6. Include a version and last-updated date

See [docs/standards/prompt-standards.md](docs/standards/prompt-standards.md) for the full prompt authoring guide.

---

## Examples

Examples in `examples/` represent real operational patterns — not tutorials and not hypothetical demonstrations. They show what a workflow produces in practice.

Examples must:
- Be drawn from or representative of actual usage
- Redact sensitive information (credentials, PII, internal system names, production data)
- Include a brief context note explaining what situation the example represents
- Show the validation and correction steps, not just the AI output

Examples must not:
- Explain what AI is or how LLMs work
- Be structured as step-by-step tutorials for beginners
- Cherry-pick unusually good AI outputs that don't reflect typical quality
- Contain fabricated AI capability demonstrations

---

## Review process

All contributions require review from a repository maintainer before merge. The reviewer checks:

1. **Template compliance** — does the file follow the correct template structure?
2. **Validation gate presence** — does the workflow include explicit human review gates at each step?
3. **Language precision** — does the content avoid exaggerated AI capability claims?
4. **Completeness** — are input contracts, output contracts, and failure modes defined?
5. **Practical accuracy** — does the workflow reflect how the task is actually performed?

---

## What not to contribute

- Prompts without workflow context
- AI capability demonstrations
- Benchmarks or model comparisons
- Tutorials explaining how to use AI tools
- Content positioning AI as a replacement for engineering judgment
- Workflows with no human review gates

---

## Categories

Current workflow categories:

| Category | Covers |
|---|---|
| `engineering/` | Code review, debugging, architecture, incidents, technical decisions |
| `delivery/` | Ticket refinement, documentation, onboarding |
| `communication/` | Async updates, status reports, technical writing |
| `operations/` | Observability investigation, postmortems |

To propose a new category, open an issue with a description of the gap and at least two concrete workflows that would belong to it. Categories should not be created for a single workflow.

---

## Workflow lifecycle

Workflows are living documents. When the tooling, process, or system context they reference changes:

1. Update the workflow to reflect current practice
2. Update the corresponding prompt files if the AI input changes
3. Note significant changes in the prompt's changelog section
4. If a workflow is retired or superseded, mark it deprecated with a link to the replacement — do not delete historical workflows

Outdated workflows that no longer reflect real practice are more harmful than no workflow. Maintainers should flag drift when they observe it.
