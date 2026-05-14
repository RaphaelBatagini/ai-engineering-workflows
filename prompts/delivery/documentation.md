# Documentation prompt

> **Category:** delivery
> **Version:** v1.0
> **Last updated:** 2026-05-13
> **Related workflow:** [`workflows/delivery/documentation.md`](../../workflows/delivery/documentation.md)

---

## Purpose

Generates a structured documentation draft from source material (code, ADRs, existing docs, runbooks). The output is a starting point for engineer review and completion — not a finished artifact. The engineer validates accuracy, fills gaps that only system knowledge can fill, and owns the final content.

---

## Input contract

| Input | Required | Description |
|---|---|---|
| `documentation_type` | Required | What kind of documentation to produce. Options: `service-overview`, `runbook`, `api-reference`, `architecture-decision-record`, `onboarding-guide`, `change-log`. This scopes the output structure. |
| `audience` | Required | Who will read this. Examples: "new engineers onboarding to this service", "on-call engineers responding to alerts", "external API consumers", "senior engineers reviewing a design decision". Audience drives tone, assumed knowledge, and level of detail. |
| `source_material` | Required | The raw input: code, existing docs, ADR drafts, runbook notes. Paste inline or provide file references. The quality of generated documentation is bounded by the quality and completeness of source material. |
| `scope` | Required | What to include and explicitly what to exclude. Without scope boundaries, the model will make coverage decisions that may not match your intent. |
| `known_gaps` | Optional | Areas where source material is incomplete or missing. Helps the model flag placeholders rather than generate unverifiable content. |

---

## Prompt

```
You are generating a documentation draft from provided source material. Your role is to structure and synthesize the source material into the specified format. The engineer will validate accuracy, add tacit knowledge, and complete any gaps.

**Documentation type:**
[documentation_type]

**Audience:**
[audience]

**Scope (include):**
[scope — what to cover]

**Scope (exclude):**
[scope — what to explicitly leave out]

**Source material:**
[source_material]

**Known gaps in source material:**
[known_gaps — or "None identified upfront"]

---

Generate a documentation draft structured for the specified documentation type and audience.

For each section you cannot substantiate from the source material, insert a placeholder:
> [NEEDS ENGINEER INPUT: brief description of what's missing and why it matters]

Do not fabricate technical details. If you are uncertain about a specific claim, flag it:
> [VERIFY: this claim is inferred from [source]; confirm before publishing]

Format the output as production-ready markdown. Use headers, tables, and code blocks where appropriate for the documentation type. Do not include a meta-commentary section about what you generated — the output should be the document itself, ready for the engineer to review and complete.
```

---

## Output contract

The model should return a markdown document matching the structure appropriate for the specified documentation type. The document should:
- Be written for the stated audience (tone, assumed knowledge, level of detail)
- Stay within the stated scope
- Include `[NEEDS ENGINEER INPUT: ...]` placeholders where source material is insufficient
- Include `[VERIFY: ...]` flags on inferred claims
- Contain no meta-commentary (the model should not describe what it generated)

---

## Validation criteria

- [ ] Every technical claim is traceable to the provided source material or flagged for verification
- [ ] Placeholders appear where source material is genuinely insufficient — not as a substitute for synthesis where source material was adequate
- [ ] The audience assumption is correct — the level of detail and assumed knowledge matches the intended reader
- [ ] Scope boundaries were respected — the document doesn't cover areas marked as out-of-scope
- [ ] No command, query, or code example is included without being verified against the actual system

---

## Failure modes

| Failure | How to identify | What to do |
|---|---|---|
| Fabricated technical details | Commands, API paths, configuration keys, or metrics names in the output that don't appear in source material and aren't flagged for verification | Cross-reference all technical specifics against source; reject and re-run with: "Flag any claim not directly supported by the source material with [VERIFY]." |
| Wrong audience calibration | Output is written at the wrong level — too basic for experienced engineers or assumes too much for new engineers | Re-run with more explicit audience description: "Assume the reader has [X years of experience / has never seen this codebase / etc.]" |
| Scope bleed | Document covers areas explicitly excluded from scope | Enforce scope in re-run: "Do not include [excluded area]. If source material references it, note it briefly and move on." |
| Missing placeholder insertion | Sections on topics not covered in source material are written as if the model has knowledge it doesn't | Compare document structure against source material; re-run with: "Insert [NEEDS ENGINEER INPUT] for every section where source material is absent or insufficient." |
| Meta-commentary in output | Output begins with "Here is the documentation I generated..." or ends with "Let me know if you'd like changes" | Strip framing; re-run with explicit instruction: "Return only the documentation. No introduction, no meta-commentary, no closing offers." |

---

## Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-05-13 | Initial version |
