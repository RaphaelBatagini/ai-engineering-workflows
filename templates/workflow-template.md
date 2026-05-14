# Workflow Template

Copy this file to `workflows/<category>/<workflow-name>.md` and fill in all sections. Do not omit sections — if a section doesn't apply, note why briefly rather than deleting it.

---

# [Workflow Name]

> **Category:** [engineering | delivery | communication | operations]
> **Typical duration:** [X–Y minutes]
> **Tools:** [Claude Code | Claude.ai | GitHub | Jira | DataDog | etc.]
> **Related prompt:** [`prompts/[category]/[prompt-file].md`](../../prompts/[category]/[prompt-file].md)

---

## Context

[1–2 paragraphs. What problem does this workflow solve? When does it apply? What does AI contribute at a high level and what does the engineer contribute? No filler — if a sentence doesn't add information, remove it.]

---

## When to use this workflow

- [Specific trigger condition or scenario]
- [Specific trigger condition or scenario]
- [Specific trigger condition or scenario]

## When not to use this workflow

- [Anti-pattern, out-of-scope scenario, or case where a different workflow applies]
- [Another case where the overhead exceeds the benefit]

---

## Prerequisites

- [ ] [Required artifact or context — be specific]
- [ ] [Required access or tooling]
- [ ] [Required knowledge or baseline understanding]

---

## Steps

### Step 1: [Step name — use a verb phrase: "Assemble context," "Generate hypotheses," "Validate output"]

**AI role:** [What the model is asked to produce — or "None" if this is a human-only step]
**Human role:** [What the engineer does, contributes, validates, or decides]

[Step description. What specifically happens in this step. What the engineer provides to the model. What the model produces. What the engineer does with the output.]

> **Validation gate:** [Specific condition that must be true before proceeding. Be concrete — "you can explain X to a colleague" or "criteria A, B, C are satisfied." Validation gates are not optional.]

---

### Step 2: [Step name]

**AI role:** [What the model is asked to do]
**Human role:** [What the engineer does]

[Step description.]

> **Validation gate:** [What must be true before proceeding.]

---

### Step N: [Step name]

[Continue for all steps. Minimum 3 steps for a meaningful workflow. Remove this placeholder line.]

---

## Output format

[Describe the expected final artifact: what type it is (code, document, analysis, ticket update, PR review), its structure, and where it lives.]

---

## Quality checklist

Before accepting this workflow's output:

- [ ] [Specific, verifiable criterion — something you can check, not "the output is good"]
- [ ] [Specific, verifiable criterion]
- [ ] [Specific, verifiable criterion]
- [ ] [Specific, verifiable criterion]

---

## Known limitations

- **[Limitation name]:** [Description of the limitation and a mitigation or workaround if one exists.]
- **[Limitation name]:** [Description of the limitation.]

---

## Related workflows

- [`workflows/[category]/[workflow].md`]([workflow].md) — [one-line description of when to use it instead of or alongside this workflow]
