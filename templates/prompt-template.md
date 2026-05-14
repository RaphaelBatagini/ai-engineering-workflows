# Prompt Template

Copy this file to `prompts/<category>/<prompt-name>.md` and fill in all sections. Every section is required. See [docs/standards/prompt-standards.md](../docs/standards/prompt-standards.md) for authoring guidance.

---

# [Prompt Name]

> **Category:** [engineering | delivery | communication | operations]
> **Version:** v1.0
> **Last updated:** [YYYY-MM-DD]
> **Related workflow:** [`workflows/[category]/[workflow-file].md`](../../workflows/[category]/[workflow-file].md)

---

## Purpose

[One sentence: what this prompt produces and for what purpose. Example: "Generates a ranked hypothesis list for debugging a described failure, with confirmation and elimination strategies for each hypothesis."]

---

## Input contract

Provide the following before running this prompt. Missing required inputs are the primary cause of low-quality outputs — do not skip this step.

| Input | Required | Description |
|---|---|---|
| `[input_name]` | Required | [What it is, why the model needs it, and what format to provide it in] |
| `[input_name]` | Required | [What it is and why] |
| `[input_name]` | Optional | [What it is and when to include it] |

---

## Prompt

```
[Static instruction text — what you want the model to do and how]

[PLACEHOLDER: describe what context to paste here — e.g., "paste the PR diff"]

[More static instructions if needed]

[PLACEHOLDER: second variable input if applicable]

Output format:
[Static specification of the expected response structure]

[Any constraints — "Do not X," "Only address Y," "If you cannot answer from the context provided, say so explicitly"]
```

---

## Output contract

Expected response structure:

1. **[Section name]:** [What this section contains and what it looks like when correct]
2. **[Section name]:** [What this section contains]
3. **[Section name]:** [What this section contains]

---

## Validation criteria

Evaluate the output against these criteria before accepting it:

- [ ] [Specific, verifiable quality criterion]
- [ ] [Specific, verifiable quality criterion]
- [ ] [Specific, verifiable quality criterion]

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| [Failure mode name] | [Observable signal in the output that indicates this failure] | [Corrective action — revise prompt, add context, change approach] |
| [Failure mode name] | [Observable signal] | [Corrective action] |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | [YYYY-MM-DD] | Initial version |
