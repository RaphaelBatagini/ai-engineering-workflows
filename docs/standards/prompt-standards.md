# Prompt Standards

This document defines how prompts in this repository are authored, structured, reviewed, and maintained.

---

## Why prompts are treated as engineering artifacts

A prompt is a specification. It defines what context is required, what the model is asked to produce, and what success looks like. A poorly specified prompt produces outputs that require more correction time than the workflow saves. A well-specified prompt produces consistent, reviewable outputs that accelerate real work.

Prompts in this repository are:
- Versioned (via git history and in-file changelog)
- Peer-reviewed before merge
- Maintained when workflows or referenced tools evolve
- Documented with input contracts, output contracts, and known failure modes

This is the same discipline applied to any engineering specification.

---

## Prompt structure

Every prompt follows this structure (see the full template at [`templates/prompt-template.md`](../../templates/prompt-template.md)):

```
# [Prompt Name]

> Version, category, related workflow

## Purpose
## Input contract
## Prompt
## Output contract
## Validation criteria
## Failure modes
## Changelog
```

---

## Authoring principles

### 1. Define context explicitly

Do not write prompts that depend on implicit context. If the model needs to know the language, the framework, the team's review standard, or the incident severity — it must be in the prompt.

**Weak:**
> "Review this code and identify issues."

**Better:**
> "Review this Go code against the following criteria: [list]. The code is part of a payment processing service. Focus on correctness and error handling in the failure path. Do not comment on style unless it creates functional ambiguity."

### 2. Specify the output structure

Tell the model exactly what format is expected. This makes outputs reviewable and reduces variance across sessions.

**Weak:**
> "Summarize the incident."

**Better:**
> "Produce a structured incident summary with: (1) chronological timeline, (2) confirmed root cause or leading hypothesis with supporting evidence, (3) impact scope, (4) mitigations applied, (5) open questions requiring follow-up."

### 3. Set explicit scope boundaries

Define what the model should not do. This prevents scope expansion and unrequested recommendations.

Include constraints such as:
> "Do not make recommendations outside the scope of [X]."
> "If you cannot answer from the provided context, say so explicitly rather than speculating."

### 4. Use placeholders for variable inputs

Prompts should be reusable across sessions. Variable inputs are clearly marked as placeholders:

```
[DIFF: paste the PR diff here]
[TICKET: paste the Jira ticket content here]
[CONTEXT: brief description of the service and its role]
```

This makes it unambiguous what the engineer must supply and prevents accidental omission.

### 5. Write for repeatability

The goal is consistent output quality across multiple engineers running the same prompt in different sessions. Before finalizing a prompt, run it across two or three realistic inputs. If outputs vary significantly in quality or structure, the prompt needs more constraint.

---

## Prompt scope and length

- Keep prompts focused on a single, well-defined task
- If a prompt exceeds approximately 300 words of instructions, split it into sub-prompts mapped to distinct workflow steps
- Avoid combining multiple distinct requests in a single prompt: "analyze the code, then write tests, then document it" — this produces outputs that are harder to validate and tend to be shallow on each task

---

## Variable inputs vs. static instructions

| Component | Example | Treatment |
|---|---|---|
| Instructions | "Identify logic errors and security issues" | Static — part of the specification |
| Context | The code diff, the ticket body, the log excerpt | Variable — supplied per session |
| Format specification | "Output as a numbered list with severity labels" | Static — part of the output contract |
| Constraints | "Do not recommend framework changes" | Static — part of the specification |

Variable inputs are marked with `[PLACEHOLDER]` syntax in the prompt body.

---

## Review requirements for new prompts

Before a prompt is merged, the reviewer checks:

1. **Input contract completeness** — are all required inputs identified and described?
2. **Output contract clarity** — is the expected output structure unambiguous?
3. **Validation criteria specificity** — can an engineer evaluate the output against the criteria without re-reading the full workflow?
4. **Failure mode coverage** — are the most common bad outputs identified with recognition signals and corrective actions?
5. **Prompt precision** — does the prompt tell the model what not to do, not just what to do?

---

## Maintenance

Prompts require updates when:

- The underlying workflow changes
- Repeated failure modes are observed in operational use that the current prompt doesn't prevent
- Referenced tools change (e.g., DataDog query syntax, Jira field structure, API changes)
- Output quality degrades consistently across sessions

When updating a prompt: increment the version, describe the change in the changelog section, and verify the input/output contracts still match the corresponding workflow.
