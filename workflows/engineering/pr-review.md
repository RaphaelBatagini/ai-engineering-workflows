# PR review workflow

> **Category:** engineering
> **Typical duration:** 15–30 minutes
> **Tools:** Claude Code or Claude.ai; GitHub (direct or via MCP)
> **Related prompt:** [`prompts/engineering/pr-review.md`](../../prompts/engineering/pr-review.md)

---

## Context

This workflow structures the use of AI analysis in pull request reviews. It is designed for complex or high-risk PRs where a systematic review pass adds concrete value — large diffs, changes to critical paths, security-relevant changes, or code in unfamiliar areas.

AI contributes a structured first-pass analysis covering logic, edge cases, security, and scope alignment. The engineer validates that analysis against system knowledge and authors the actual review. The approval decision is always the engineer's.

---

## When to use this workflow

- The diff is large (>200 lines of substantive change) and you want a structured analysis before beginning manual review
- The PR touches an area of the codebase unfamiliar to you
- The PR has security implications: authentication, authorization, data handling, external API changes
- The PR modifies a critical path where subtle bugs are high-cost
- You want systematic edge case coverage before approving

## When not to use this workflow

- Small, focused PRs where manual review is faster and the context is fully clear
- Trivial changes: config updates, dependency version bumps with no behavior change, comment-only edits
- As a substitute for understanding the change — this workflow accelerates review, it does not replace it

---

## Prerequisites

- [ ] The PR diff is accessible (GitHub UI, `git diff main...HEAD`, or GitHub MCP)
- [ ] The PR description is written and readable
- [ ] The linked ticket or issue is accessible
- [ ] You have a working understanding of the service or component being changed

---

## Steps

### Step 1: Assemble review context

**AI role:** None
**Human role:** Collect and organize all inputs before analysis

Collect:
1. The full PR diff
2. The PR title, description, and any existing review comments
3. The linked ticket: problem statement and acceptance criteria
4. Relevant context not visible in the diff: is this a critical path? Does it have external consumers? Are there known constraints the author may not be aware of?

Document any context the model cannot infer from the diff: team conventions, known technical debt, constraints, or the reasoning behind the approach.

> **Validation gate:** All inputs are assembled before running the analysis. A PR review without ticket context is incomplete — the model cannot assess scope alignment without the acceptance criteria.

---

### Step 2: Run AI-assisted review analysis

**AI role:** Analyze the diff for logic correctness, edge cases, security considerations, error handling, test coverage, and scope alignment with the linked ticket
**Human role:** Provide the assembled context; observe what the model surfaces

Use the [PR review prompt](../../prompts/engineering/pr-review.md) with the full assembled context.

Expected output:
- Scope alignment: does the diff implement what the ticket describes? Is anything missing or out of scope?
- Logic and correctness: specific issues with file and function references
- Edge cases: unhandled input conditions or concurrency scenarios
- Security considerations: issues specific to this diff
- Error handling gaps
- Test coverage observations

> **Validation gate:** Do not treat the model's analysis as a complete review. It is a structured first pass. The model cannot know your system topology, team conventions, or historical context. Proceed to Step 3 to apply that knowledge.

---

### Step 3: Validate the analysis

**AI role:** None
**Human role:** Apply system knowledge to validate, supplement, and curate the model's analysis

Work through the model's output item by item:

For each flagged issue:
- Is this actually an issue in this codebase, or a generic concern that doesn't apply here?
- Is the severity assessment accurate given the context and operational risk?
- Does the suggested direction align with the team's patterns and constraints?
- Is there system knowledge the model couldn't have that changes the assessment?

Add issues the model missed based on your domain knowledge. Remove issues that are not real given the system context. Adjust severity based on actual operational impact.

> **Validation gate:** You can explain and defend every item in the validated analysis from first principles. Remove anything you cannot explain or do not understand well enough to defend.

---

### Step 4: Author review comments

**AI role:** Optional — can draft comment text from validated analysis items
**Human role:** Author or revise every comment before posting

If you use AI to draft comment text from the validated items:
- Provide the specific, validated analysis item as input
- Review every draft for tone, precision, and accuracy
- Revise before posting — the comment is yours

Review comments must:
- Reference the specific file and line or function
- Explain the concern, not just flag it
- Propose or suggest a direction where possible
- Clearly distinguish blocking issues from non-blocking observations

> **Validation gate:** Every comment you post reflects your professional judgment. You have read and validated each comment — not just accepted an AI draft.

---

### Step 5: Submit the review

**AI role:** None
**Human role:** Make the approval decision and submit

The approval decision is yours. Consider:
- Are all blocking issues addressed or tracked?
- Is the change safe to merge given its risk profile?
- Have non-blocking observations been communicated appropriately?

Submit the review via GitHub.

---

## Output format

The final output is your GitHub PR review: inline comments, a summary comment, and an approve / request changes / comment decision.

---

## Quality checklist

Before submitting the review:

- [ ] Every comment references a specific file and function — no generic observations
- [ ] Blocking vs. non-blocking issues are explicitly distinguished in the review
- [ ] The approval decision reflects your actual assessment of the change
- [ ] Any model-flagged items you discarded were consciously evaluated and rejected, not ignored
- [ ] Context the model could not have — team conventions, historical decisions, operational context — has been applied

---

## Known limitations

- **Context window on large diffs:** Very large diffs (>1000 lines) may need to be reviewed in logical sections. The model may miss interactions between sections when processing a split diff. Identify cross-cutting concerns manually.
- **Tacit knowledge gap:** The model cannot know your team's unwritten conventions, historical architectural decisions, or real production load patterns. Step 3 exists specifically to apply this context.
- **Generic flag noise:** The model will surface generic concerns that don't apply to your specific codebase. Expect 20–30% of flags to be discarded in Step 3 for mature services. This is expected behavior, not a prompt failure.
- **Novel API and library versions:** The model may not have current knowledge of very recent library releases or internal libraries. Validate references to specific APIs independently.

---

## Related workflows

- [Code Review](code-review.md) — deeper quality review not tied to a specific PR
- [Architecture Review](architecture-review.md) — for PRs introducing significant design decisions
- [Debugging](debugging.md) — when reviewing a bug fix and needing to validate the fix logic independently
