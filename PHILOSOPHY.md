# Engineering Philosophy

This document defines the principles that govern how AI is used in engineering workflows within this repository.

---

## Core Stance

AI is an engineering accelerator. It compresses time on well-defined tasks, surfaces information faster, and generates structured first drafts that engineers refine. It does not replace engineering judgment, system knowledge, or accountability.

Every workflow in this repository is designed from that position.

---

## Principles

### 1. Human-in-the-loop is a design constraint, not a disclaimer

AI-assisted workflows must be designed so that human review is structurally required — not just recommended. This means:

- Every workflow has defined validation gates where engineer review happens before the process continues
- Output quality criteria are explicit, so review is consistent across engineers and sessions
- The engineer, not the model, is responsible for the final artifact

This is not a hedge against AI capability. It is sound engineering practice applied consistently.

### 2. Prompt quality is an engineering concern

A prompt is a specification. A vague specification produces unpredictable output. A well-structured prompt with a clear input contract, output contract, and defined failure modes produces consistent, reviewable output.

Prompts in this repository are versioned, peer-reviewed, and maintained with the same discipline as code. A prompt that consistently produces low-quality outputs is a defect, not a limitation to work around manually.

### 3. Validation is not optional and not informal

AI-generated outputs — code, analysis, documentation, investigation summaries — are subject to review. That review has explicit criteria defined per workflow. "It looks fine" is not a quality gate.

This matters operationally: a poorly reviewed AI output in a debugging session or incident investigation has the same blast radius as a poorly reviewed code change. The source of an artifact does not change the review standard it requires.

### 4. Workflow design precedes tool selection

The value of AI-assisted engineering comes from sound process design, not from the specific model or toolchain. Workflows are designed to be tool-informed — they reference Claude Code, MCP, GitHub, Jira, DataDog — but the underlying process logic is independent of any specific vendor.

When tools change, workflows adapt. The engineering principles do not.

### 5. Operational reliability over coverage

A small set of consistently applied, well-designed workflows delivers more value than a large library of inconsistently used ones. This repository prioritizes depth over breadth: each workflow covers edge cases, failure modes, and quality criteria rather than being a shallow template.

Adding a new workflow that will not be maintained or consistently applied is worse than not having it.

### 6. Precision in how we talk about AI

This repository does not make exaggerated claims about AI capability. Language is deliberate:

- "AI-assisted" not "AI-powered" or "AI-driven"
- "accelerates" not "replaces" or "automates"
- "generates a structured first draft" not "writes the documentation"
- "supports the decision" not "makes the recommendation"

This precision matters for engineering culture. Teams that are precise about what AI does and does not do build more reliable workflows than teams operating on vague assumptions about AI capability. Overconfident assumptions about AI output quality lead directly to skipped validation steps, which is where most AI-assisted workflow failures originate.

---

## What this repository optimizes for

- Predictable, consistent output quality across AI-assisted engineering tasks
- Reduced cognitive overhead on well-defined, repeatable tasks
- Faster first drafts with reliable review workflows
- Shared language and standards across engineering teams adopting AI
- Operational reliability: workflows that work consistently, not just occasionally

## What this repository does not optimize for

- Maximizing AI usage across all engineering tasks
- Demonstrating AI capabilities
- Replacing established engineering processes with AI-native alternatives
- Tracking or adopting new AI tool releases

---

## On failure modes at the workflow level

The most common failure modes in AI-assisted engineering workflows are not model quality issues — they are process issues:

1. **Insufficient input context.** The engineer provides a vague description and receives a vague analysis. The model produced the best output available from the input; the input was the defect.

2. **Skipped validation.** The engineer accepts output that "looks right" without working through the quality criteria. This is where hallucinated facts, generic observations, and miscalibrated confidence get propagated into real artifacts.

3. **Scope drift.** The model produces output beyond what was asked — additional recommendations, unrequested redesigns, or adjacent analyses. The engineer accepts the full output without curating it to the actual scope.

4. **Tacit knowledge gaps.** The model cannot know team conventions, historical decisions, operational context, or organizational constraints. The engineer knows these things but doesn't apply them during review.

These failure modes are all engineer-side process failures, not model limitations. The workflows in this repository are designed to prevent them structurally.

---

## On AI model selection

This repository is not prescriptive about which model to use. Examples reference Claude and Claude Code because they are used in the workflows that informed this repository's design. The prompt specifications and workflow structures are portable — they can be adapted for other models.

Model selection is an engineering decision that belongs to each team based on their toolchain, data classification requirements, and operational context.
