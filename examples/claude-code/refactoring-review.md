# Example: Refactoring review — AI-assisted PR review with scope validation

This example shows a complete PR review session using the [PR review workflow](../../workflows/engineering/pr-review.md) and the [PR review prompt](../../prompts/engineering/pr-review.md), applied to a refactoring PR.

Refactoring PRs have a distinct review dynamic: they're larger than feature changes, often touch more files, and require verifying that behavior is preserved rather than that new behavior is correct. This example shows how AI-assisted review handles this effectively — and where the engineer must stay alert for the specific failure modes that refactoring reviews introduce.

> **Note:** Names, specifics, and file paths are representative. Sensitive details have been redacted.

---

## Situation

**Service:** `user-service` — handles user profile, preferences, and session management
**PR:** `refactor: extract session management from UserService into SessionManager`
**Size:** 18 files changed, ~850 lines
**Description:** The PR extracts session-related logic from a large `UserService` class into a dedicated `SessionManager` class. The PR description states this is a pure refactor — no behavior changes.

**Reviewer context:** The reviewing engineer is familiar with the service but not the session management code paths in detail.

---

## Step 1: Assemble context

The reviewer confirms the stated scope: this should be a pure structural refactor. The reviewer identifies the key validation question before running AI analysis: **is any session behavior changing, or is this a clean extraction?**

Context assembled:
- Full PR diff (18 files)
- PR description: "Extracts session lifecycle management from UserService. No API contract changes. All session state previously in UserService.sessions is now managed by SessionManager."
- Ticket context: "Tech debt cleanup — UserService has grown too large. No feature work in this ticket."
- Service context: "Session management is on the hot path — called on every authenticated request."

---

## Step 2: Run AI analysis

**Prompt submitted to Claude Code:**

The full PR review prompt with the above inputs, with `review_focus` set to: "Scope validation (is this truly a pure refactor?), logic correctness (does SessionManager replicate UserService session behavior exactly?), and any reliability issues introduced by the extraction."

**Model output (representative):**

```
**1. Scope alignment**

The PR description claims this is a pure structural refactor with no behavior changes.
The diff is consistent with this claim in most areas, with two exceptions that require
engineer validation:

Exception 1 — Session expiry comparison (UserService.go line 142 vs SessionManager.go line 89):
UserService used `time.Now().After(session.ExpiresAt)` for expiry checks.
SessionManager uses `session.ExpiresAt.Before(time.Now())`.
These are logically equivalent but the change was not noted in the PR description.
Confirm intent.

Exception 2 — Session cleanup on logout (UserService.go line 203 vs SessionManager.go line 156):
UserService called `deleteSession` then `clearSessionCookie` sequentially.
SessionManager calls these in the same order, but `clearSessionCookie` is now called
inside a deferred function. This changes the execution order if `deleteSession` panics
— `clearSessionCookie` will run in the panic recovery path under SessionManager but
not under UserService. This is a behavior difference in the error path.

---

**2. Logic and correctness**

The extraction preserves UserService session logic correctly across the non-error paths
reviewed. Session creation, lookup, and refresh logic in SessionManager matches the
UserService implementation.

Finding: Concurrent session lookup (SessionManager.go line 201)
UserService used a read lock for session lookups (`mu.RLock()`). SessionManager uses
a full write lock (`mu.Lock()`) for the same operation. This is functionally correct
but degrades throughput on a path called on every authenticated request. Given the
"session management is on the hot path" context, this may not be an intentional
change.

---

**3. Edge cases**

Finding: Nil session handling (SessionManager.go line 78)
UserService returned an error when a nil session was passed to `RefreshSession`.
SessionManager panics with a nil pointer dereference on the same input. The nil case
is guarded elsewhere before calling the method, but the removal of the guard inside
the method makes the behavior of SessionManager less defensive.

---

**4. Security**

No security concerns identified. Session token generation and storage patterns are
unchanged. Cookie flags (HttpOnly, Secure, SameSite) are set identically.

---

**5. Error handling**

The deferred `clearSessionCookie` behavior noted in Scope Alignment is the primary
error handling finding. If `deleteSession` returns an error (not just panics), the
original UserService returned the error and skipped `clearSessionCookie`. SessionManager's
deferred implementation will call `clearSessionCookie` regardless of whether
`deleteSession` returned an error. This changes logout behavior when session deletion
fails — the cookie is now cleared even if server-side session deletion failed.

---

**6. Test coverage**

The test suite is updated to cover SessionManager. Tests exist for the happy path
and session expiry. Not observed: tests for the nil session input case (see Edge Cases),
and no test for the logout error path (see Error Handling).

---

**7. Observations**

- SessionManager struct is exported but several internal methods are also exported
  (GetSession, DeleteSession) with no documented intent to use them outside the package.
  If these are not intended to be part of a public API, they could be unexported.
- Error messages in SessionManager use a different format than the rest of user-service
  ("session error: ..." vs "userservice: ..."). Minor inconsistency.
```

---

## Step 3: Validate the analysis

The reviewer goes through each finding:

**Exception 1 (expiry comparison):** Verified — `After` vs `Before` is logically equivalent. The reviewer confirms this is a style difference, not a behavior change. No action needed, but the reviewer adds a comment asking the author to note it in the PR description for clarity.

**Exception 2 (deferred clearSessionCookie):** **Confirmed real finding.** The reviewer traces the logout path and confirms the behavior change. Under the old code, if `deleteSession` failed, the user's cookie was not cleared (belt-and-suspenders: don't log the client out if server-side cleanup failed). Under SessionManager, the cookie is always cleared regardless. The reviewer flags this for discussion with the author — it may be intentional (clear the cookie always to avoid orphaned sessions) but it's a behavior change that should be explicit.

**Concurrent session lookup (write lock vs. read lock):** **Confirmed finding.** The reviewer checks the original lock type and confirms `mu.RLock()` was used. This is a regression for read-heavy session lookup throughput. The reviewer adds a comment requesting the author change to `mu.RLock()` to match the original.

**Nil session panic:** The reviewer checks where `RefreshSession` is called. The guard is present in all three call sites. The reviewer judges the risk as low but requests the nil guard be restored inside `RefreshSession` as a defensive measure — a future caller might not add the guard.

**Test coverage gaps:** The reviewer requests tests for the nil input case and the logout error path before approval.

**Observations:** The reviewer notes the exported methods and will raise in a follow-up ticket rather than blocking the PR.

---

## Step 4: Author review comments

The reviewer submits the following blocking comments:

1. **Behavior change — logout error path:** "This is a behavior change, not a refactor. The old code preserved the cookie if `deleteSession` failed. The new code clears it regardless due to the deferred call. Is this intentional? If so, it needs to be documented in the PR description and the commit message."

2. **Lock regression:** "`mu.Lock()` on session lookup is a regression from the original `mu.RLock()`. Session lookup is on the hot path. Please revert to a read lock."

3. **Nil session guard:** "The nil guard was removed inside `RefreshSession`. Please restore it — the method should not panic on nil input regardless of what callers currently do."

4. **Missing tests:** "Please add tests for: (a) nil session input to `RefreshSession`, (b) logout when `deleteSession` returns an error."

And one non-blocking comment:
5. "The expiry check changed from `time.Now().After(...)` to `ExpiresAt.Before(...)` — these are equivalent, but worth noting in the PR description since reviewers will flag it."

---

## Notes on workflow value

**What the AI analysis added:**
- The lock regression (write lock replacing read lock) is the kind of finding that's easy to miss in an 850-line diff — a reviewer scanning for behavior changes is less likely to catch a performance regression. The model identified it because it's explicitly looking at each method's implementation.
- The deferred `clearSessionCookie` behavior change was caught and correctly characterized as a semantic difference, not just a structural one.
- The test coverage gaps were identified systematically — the reviewer's attention was on logic correctness, and the test coverage check might have been lighter without the structured output.

**What required engineer judgment:**
- The reviewer had to decide whether the nil session risk was acceptable (low given the guarded call sites) — the model couldn't make that call.
- The logout behavior change required a domain judgment: "does it make more sense to clear the cookie even when server-side deletion fails?" That depends on the product's security posture, which the model doesn't know.
- The reviewer decided to raise the exported methods in a follow-up rather than blocking — a judgment about PR scope and team norms.

**Refactoring-specific note:**
Refactoring PRs benefit particularly from the "scope alignment" section of the PR review prompt. The most important question for a refactoring PR is not "is the new code correct?" but "does the new code do the same thing as the old code?" Framing the review focus around scope validation surfaces behavioral differences that a standard correctness review might miss.
