# Example: Jira MCP — ticket refinement workflow

> **Workflow:** [Ticket Refinement](../../workflows/delivery/ticket-refinement.md)
> **Prompt:** [Ticket refinement prompt](../../prompts/delivery/ticket-refinement.md)
> **Tool:** Claude Code with Jira MCP
> **Context:** Representative example showing Jira MCP in a pre-refinement session. Ticket content and project context generalized.

---

## What changes with Jira MCP

Without MCP, the engineer copies the ticket content from Jira into the prompt. With Jira MCP, the model can:

- Fetch the full ticket including all fields, description, comments, and attachments
- Follow linked tickets (dependencies, parent epics, related issues)
- Read the epic-level context to understand the broader feature scope
- Check sprint context and other tickets in the same sprint for scope consistency

The refinement workflow and validation steps are identical. MCP enables analysis of the full ticket context including linked issues, which often contains scope-defining information that engineers don't always copy manually.

---

## Session transcript (representative)

**Engineer prompt:**

```
Refine ticket ENG-2847 for sprint readiness.

System context: this ticket is for the notification service, which handles async email and 
push notifications. Notifications are delivered via a queue — no synchronous delivery.

Scope boundary: this ticket covers the notification preferences API endpoint only. 
The preference UI and notification template changes are separate tickets.
```

**MCP actions taken by Claude Code:**

```
Fetching ENG-2847: "User notification preferences — API endpoint"
  - Epic: ENG-2700 "User notification preferences (Phase 1)"
  - Description: [retrieved — 3 paragraphs]
  - Acceptance criteria: 2 criteria listed
  - Linked: ENG-2848 (notification preferences UI), ENG-2849 (template updates)
  - Comments: 3 (product clarification, engineering question, PM response)

Fetching epic ENG-2700 for scope context...
  - Epic description: [retrieved]
  - Phase 1 scope definition: [retrieved]
```

**Model output (abbreviated):**

```
## Acceptance criteria

Current criteria in the ticket (2):
1. "Users can update their notification preferences via API" — not testable as written. 
   Suggest: "PUT /v1/users/{id}/notification-preferences returns 200 and persists the 
   preferences when called with a valid request body"
2. "Invalid preferences are rejected" — not testable. 
   Suggest: "PUT /v1/users/{id}/notification-preferences returns 422 with an error body 
   when the request contains an unrecognized preference key"

Additional criteria not in the ticket:
3. "GET /v1/users/{id}/notification-preferences returns the current preferences with a 200 
   when the user exists" — the ticket description implies a read endpoint but no criterion 
   covers it
4. "The endpoint returns 404 when the user ID does not exist"
5. "Preference updates are reflected in subsequent notification delivery" — this may be 
   Phase 2 scope; requires PM clarification

## Ambiguities

1. The ticket description mentions "notification preferences" without defining the preference 
   schema. The PM comment in the ticket (2026-05-08) references a Figma mockup but the 
   preference keys and valid values are not specified anywhere accessible. Without the schema, 
   acceptance criteria 3 and 4 cannot be implemented.
   
   Clarifying question: What is the preference schema? Which keys are supported and what are 
   their valid values?

2. Authentication requirements are not specified. Should this endpoint require the user to be 
   authenticated as the same user whose preferences are being modified, or is admin-level 
   access also permitted?

## Edge cases

1. User ID in path does not match the authenticated user's ID — is this a 403 or a 404?
2. Partial updates: if the request body contains only some preference keys, are unspecified 
   preferences preserved or reset to defaults?
3. Concurrent updates: no mention of optimistic locking or last-write-wins behavior
4. Very large preference payload: is there a payload size limit?

## Scope observations

The epic (ENG-2700) defines Phase 1 as "API endpoint only, no UI." This ticket is correctly 
scoped. Criterion 5 above ("reflected in delivery") appears to be Phase 2 scope based on the 
epic definition.
```

---

## Engineer validation (Step 3)

**Acceptance criteria:** 
- Criteria 1 and 2 revisions: accepted — both improve testability significantly
- Criterion 3 (GET endpoint): confirmed in scope — engineer verified with PM that a read endpoint is expected
- Criterion 4 (404): accepted
- Criterion 5 (delivery reflection): confirmed out of scope for this ticket per epic definition

**Ambiguities:**
- Schema ambiguity: confirmed real. Engineer escalated to PM for a documented preference schema before sprint commit. Ticket moved to "Needs Info."
- Authentication requirements: answered by existing API security convention — same-user or admin. Engineer added to ticket description.

**Edge cases:**
- Concurrent updates: noted as out of scope with explicit "deferred — Phase 2" comment
- Partial updates: confirmed as preserve-unspecified behavior — added to acceptance criteria
- Payload size: inherited from existing API limits — noted in ticket with reference to API limits doc

---

## Notes on MCP value in this session

Jira MCP fetched the epic-level scope definition, which the engineer had not included in the manual context. This led to the correct scoping of Criterion 5 as out of scope — a judgment that required the Phase 1/Phase 2 boundary definition from the epic.

The PM comment in the ticket (referencing the Figma mockup for the preference schema) was retrieved via MCP. Without MCP, this comment might not have been included in the manual copy-paste, and the schema ambiguity might not have been surfaced until mid-sprint.
