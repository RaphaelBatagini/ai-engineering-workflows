# ai-engineering-workflows

Engineering workflows, prompt standards, and operational patterns for AI-assisted software delivery.

---

This repository is a practical reference for engineering teams integrating AI tools into real engineering processes — code review, incident investigation, architecture decisions, async communication, and delivery operations.

It is not a prompt collection. It is not an AI capabilities showcase. It is an engineering playbook.

---

## What this is

- A structured library of AI-assisted engineering workflows with explicit human review gates
- Prompt engineering standards for consistent, reviewable AI inputs
- Quality criteria and validation checklists for AI-generated outputs
- Operational examples using Claude Code, MCP-enabled toolchains, and GitHub/Jira/DataDog integrations
- A reference architecture for teams that want to adopt AI systematically, not opportunistically

## What this is not

- A collection of "useful prompts" with no process context
- An argument that AI replaces engineering judgment
- A tutorial for engineers learning what LLMs can do
- A showcase of AI capabilities or benchmark results
- Hype-driven content about AI productivity multipliers

---

## Engineering Principles

**1. AI assists; engineers decide.**
Every workflow in this repository includes explicit human review gates. AI output is always an input to an engineering decision — never the decision itself.

**2. Structured inputs produce reviewable outputs.**
Prompt quality is an engineering concern. Vague prompts produce vague outputs. This repository treats prompts as specifications with input contracts, output contracts, and failure modes.

**3. Validation is not optional.**
AI-generated outputs — code, analysis, documentation — require the same review discipline as any other engineering artifact. Quality gates are defined per workflow, not left to individual discretion.

**4. Workflows before tooling.**
The process design matters more than the specific model or tool. Workflows in this repository are tool-informed but not tool-dependent. When tools change, workflows adapt.

**5. Operational reliability over feature coverage.**
A small set of well-designed, consistently applied workflows delivers more value than a large library of inconsistently used ones.

---

## Repository Map

| Directory | Purpose |
|---|---|
| [`workflows/`](workflows/) | Process specifications for AI-assisted engineering tasks |
| [`prompts/`](prompts/) | Prompt specifications: input contracts, output contracts, failure modes |
| [`guardrails/`](guardrails/) | Review checklists, quality gates, validation criteria |
| [`docs/standards/`](docs/standards/) | Authoring standards for workflows and prompts |
| [`examples/`](examples/) | Representative artifacts from real operational patterns |
| [`templates/`](templates/) | Reusable scaffolding for new workflows, prompts, and ADRs |
| [`docs/onboarding/`](docs/onboarding/) | Getting started guide for new contributors and users |

---

## Workflow Index

### Engineering

| Workflow | Use case |
|---|---|
| [PR Review](workflows/engineering/pr-review.md) | AI-assisted pull request analysis with structured review output |
| [Code Review](workflows/engineering/code-review.md) | Systematic code quality review using AI-generated analysis |
| [Debugging](workflows/engineering/debugging.md) | Structured debugging sessions with AI hypothesis generation |
| [Architecture Review](workflows/engineering/architecture-review.md) | AI-assisted architecture analysis and decision support |
| [Incident Investigation](workflows/engineering/incident-investigation.md) | AI-accelerated root cause analysis with engineer-led conclusions |
| [Technical Decision](workflows/engineering/technical-decision.md) | Structured decision support with tradeoff analysis |

### Delivery

| Workflow | Use case |
|---|---|
| [Ticket Refinement](workflows/delivery/ticket-refinement.md) | AI-assisted acceptance criteria, scope analysis, and edge case identification |
| [Documentation](workflows/delivery/documentation.md) | Engineering documentation generation with review workflow |
| [Onboarding](workflows/delivery/onboarding.md) | AI-accelerated onboarding material generation and knowledge transfer |

### Communication

| Workflow | Use case |
|---|---|
| [Async Communication](workflows/communication/async-communication.md) | AI-assisted async update drafting, standup summaries, and status reports |

### Operations

| Workflow | Use case |
|---|---|
| [DataDog Investigation](workflows/operations/datadog-investigation.md) | AI-assisted observability analysis and alert investigation |
| [Postmortem](workflows/operations/postmortem.md) | Structured postmortem documentation with AI-assisted timeline reconstruction |

---

## Getting Started

**Using a workflow:**
1. Navigate to the relevant workflow file in [`workflows/`](workflows/)
2. Review the prerequisites and input requirements
3. Follow the steps — each step identifies the AI role, the human role, and the validation gate
4. Apply the quality checklist before accepting any output

**Using a prompt:**
1. Navigate to the corresponding prompt in [`prompts/`](prompts/)
2. Satisfy the input contract before running the prompt
3. Evaluate the output against the validation criteria defined in the prompt file
4. Consult the failure modes section if output quality is low

**Adding a new workflow:**
See [CONTRIBUTING.md](CONTRIBUTING.md) and the [workflow template](templates/workflow-template.md).

---

## Standards

- [Prompt Standards](docs/standards/prompt-standards.md) — how prompts are authored, structured, and maintained
- [Output Quality](docs/standards/output-quality.md) — what makes an AI-assisted output acceptable
- [Validation Standards](docs/standards/validation-standards.md) — review gates and approval criteria
- [Quality Gates](guardrails/quality-gates.md) — per-output-type quality criteria

---

## Philosophy

The full engineering philosophy behind this repository is in [PHILOSOPHY.md](PHILOSOPHY.md).

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for workflow and prompt authoring standards, naming conventions, and the review process for new contributions.
