# Example: GitHub MCP — PR review workflow

> **Workflow:** [PR Review](../../workflows/engineering/pr-review.md)
> **Prompt:** [PR review prompt](../../prompts/engineering/pr-review.md)
> **Tool:** Claude Code with GitHub MCP
> **Context:** Representative example of how GitHub MCP changes the context-gathering step of the PR review workflow. The workflow logic and validation requirements are unchanged.

---

## What changes with GitHub MCP

Without MCP, the engineer manually copies the PR diff, description, and ticket link from GitHub into the AI session. With GitHub MCP, the model can:

- Fetch the PR diff directly by PR number
- Read PR description and existing review comments
- Follow the linked issue to retrieve ticket context
- Check CI status and code owners

The review analysis and validation steps are identical. MCP removes the copy-paste step and reduces the risk of truncated diffs on large PRs.

---

## Session transcript (representative)

**Engineer prompt:**

```
Review PR #847 in [org]/[repo]. The linked issue is #392.

Focus on: error handling completeness and scope alignment with the issue.

Service context: this is the billing service's charge processing module — any unhandled errors in the charge path result in silent failures that are difficult to detect and recover from.
```

**MCP actions taken by Claude Code:**

```
Fetching PR #847: "feat: add retry logic to charge processor"
  - Diff: 312 lines across 4 files
  - Description: [retrieved]
  - Linked issue #392: "Charges failing silently when payment gateway times out"
  - Existing comments: 2 (author responses to CI warnings)
  - CI status: passing
```

**Model output (abbreviated):**

```
## Scope alignment

Issue #392 describes silent failures when the payment gateway times out. The PR adds retry logic 
with exponential backoff in `charge_processor.go`. The scope matches.

One acceptance criterion from the issue is not addressed: "failed charges after retry exhaustion 
should be written to the failed_charges table for reconciliation." The PR logs the failure but 
does not write to the table.

Severity: blocking — this is an explicit acceptance criterion.

## Error handling

chargeWithRetry() (charge_processor.go:67):
  The retry loop catches errors from the gateway client but does not distinguish between 
  retryable errors (timeout, 503) and non-retryable errors (400 Bad Request, 402 Payment 
  Required). The current implementation retries all errors, including non-retryable ones.
  
  Impact: unnecessary retries on permanent failures, added latency, potential duplicate charge 
  attempts depending on gateway idempotency behavior.
  
  Severity: blocking.

...
```

---

## Engineer validation (Step 3)

**Scope alignment finding — confirmed:** Engineer verified issue #392 acceptance criteria. The failed_charges table write is explicitly required. Finding is valid and blocking.

**Error handling finding — confirmed with context:** The model's analysis is correct about non-retryable errors. Engineer added additional context: the payment gateway returns 402 for both insufficient funds (non-retryable) and card temporarily declined (retryable by convention). The fix needs to check both the HTTP status and the gateway's `error_code` field, not just the HTTP status.

**Model's suggestion for the fix:** Acceptable as a starting point but needs to handle the `error_code` distinction. Engineer revised the suggestion before adding it to the review comment.

---

## Notes on MCP value in this session

GitHub MCP reduced context preparation from ~8 minutes to ~1 minute. The full PR diff (312 lines) was provided without truncation, which would have been a copy-paste risk manually.

The model's identification of the missing failed_charges write came from reading the issue's acceptance criteria directly via MCP — this connection between PR content and issue requirements is where MCP provides the clearest signal improvement over manual copy-paste (where engineers sometimes paste only the issue title, not the full acceptance criteria).
