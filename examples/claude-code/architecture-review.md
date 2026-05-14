# Example: Architecture review — service design decision with tradeoff analysis

This example shows a complete architecture review session using the [architecture review workflow](../../workflows/engineering/architecture-review.md) and the [architecture review prompt](../../prompts/engineering/architecture-review.md).

The session covers a real-style decision: choosing between synchronous vs. asynchronous processing for a billing event pipeline. The example shows how AI surfaces tradeoffs and assumptions that engineers often overlook when close to a design, and how the engineer validates and acts on the output.

> **Note:** Names, metrics, and specifics are representative. Sensitive details have been redacted.

---

## Situation

**Service:** `billing-service` — processes payment events and updates account ledgers
**Team:** 3 engineers
**Decision:** The billing service receives payment events from upstream. Currently this is synchronous: the payment service calls billing directly and waits for a response. The team is evaluating moving to an asynchronous queue-based design.

**Trigger:** After a payment service incident where latency spikes in billing caused cascading timeouts in payment checkout, the team wants to decouple the two services. The decision has implications for consistency, observability, and error handling.

---

## Step 1: Verify proposal completeness

Before running the AI analysis, the engineer assembles the inputs:

```
Decision statement:
Should billing event processing remain synchronous (direct call from payment service)
or move to asynchronous (payment service publishes to a queue; billing consumes asynchronously)?

Constraints:
- Ledger entries must be eventually consistent within 500ms of payment confirmation under normal conditions
- The billing team owns the queue consumer; the payment team owns the producer
- Team has no current operational experience with the queue infrastructure (SQS)
- No changes to the external payment API contract are possible

Options:
Option A — Keep synchronous:
- Payment service calls billing HTTP endpoint directly
- Billing processes synchronously and returns success/error
- Payment service handles billing errors inline

Option B — Asynchronous queue (SQS):
- Payment service publishes PaymentCompleted event to SQS
- Billing consumes from queue asynchronously
- Payment service considers its work done once event is published
- Billing retries on failure via SQS visibility timeout

Evaluation criteria:
1. Blast radius reduction: does this prevent billing latency from affecting payment checkout?
2. Error handling clarity: is it clear what happens when billing fails?
3. Operational complexity: what is added to the team's operational burden?
4. Consistency guarantees: can we meet the 500ms eventual consistency requirement?
```

The engineer confirms: all required inputs are present. The proposal is complete enough to run the analysis.

---

## Step 2: Run tradeoff analysis

**Prompt submitted to Claude Code:**

The full architecture review prompt with the above inputs.

**Model output (representative):**

```
**1. Constraint check**

Option A (synchronous): Viable. No constraint violations identified.

Option B (asynchronous/SQS): Viable with a caveat. The 500ms eventual
consistency requirement is achievable under normal conditions but depends
on queue consumer throughput and SQS polling configuration. This is an
assumption that requires validation, not a constraint violation. Flagged
in open questions.

---

**2. Per-option analysis**

**Option A — Synchronous**

Core tradeoffs:
- Gain: Simple failure model. If billing fails, payment service knows
  immediately and can return an error to the customer.
- Give up: Availability coupling. Billing latency and errors propagate
  directly to payment checkout. This is the pattern that caused the
  triggering incident.

Key assumptions this option relies on:
- Billing can maintain p99 latency below payment service timeout thresholds
  under peak load.
- Billing availability matches or exceeds payment service availability
  requirements.

Operational implications:
- Existing alert on billing p99 latency is already in place (from incident).
- Timeout tuning between services must be maintained across both teams.
- No new infrastructure to operate.

Failure mode: If billing latency exceeds payment service timeout, payment
checkout degrades. The triggering incident repeats. The only mitigation is
bilateral timeout tuning and billing performance work, which treats symptoms
without removing the coupling.

---

**Option B — Asynchronous (SQS)**

Core tradeoffs:
- Gain: Blast radius reduction. Billing consumer failure or latency does
  not affect payment checkout. Payment service's critical path ends at
  queue publish.
- Give up: Synchronous error visibility. If billing fails to process an
  event, payment service does not learn about it in the customer request
  path. Failed events must be surfaced through dead-letter queues and
  separate alerting.

Key assumptions this option relies on:
- The billing team can operate SQS reliably with no current SQS experience.
- Consumer throughput is sufficient to process events within 500ms under peak load.
- Dead-letter queue monitoring is implemented before the queue is put into production.

Operational implications:
- New infrastructure: SQS queue, dead-letter queue, consumer scaling policy.
- New alerting required: dead-letter queue depth, consumer lag, processing
  error rate.
- A billing failure is now an async alert rather than a synchronous error.
  On-call runbooks must be updated.
- A payment event may be in the queue but not yet processed — the system
  is in a valid intermediate state that operations tooling must handle.

Failure mode: If dead-letter queue monitoring is not implemented, billing
failures become silent. A backlog of unprocessed events accumulates without
triggering an alert. This is a worse failure mode than Option A's synchronous
errors if the monitoring is not in place first.

---

**3. Comparative assessment**

| Criterion | Option A | Option B |
|---|---|---|
| Blast radius reduction | Does not address coupling — incident repeats | Decouples billing from checkout critical path |
| Error handling clarity | Clear — billing errors surface in checkout | Requires new dead-letter and alerting patterns |
| Operational complexity | Low (no new infrastructure) | Medium-high (new infra, new monitoring, new runbooks) |
| Consistency guarantees | Synchronous — strong | Eventually consistent within 500ms under normal conditions; requires validation |

Option B addresses the stated problem (blast radius from the triggering incident)
but transfers operational complexity onto a team with no current SQS experience.
The risk of silent billing failures under Option B is higher than the risk of
repeat incidents under Option A if monitoring is not implemented before cutover.

---

**4. Open questions and validation gaps**

1. Can the billing consumer achieve 500ms p99 processing under peak load? This
   requires load testing with realistic event volumes before committing to Option B.

2. What is the team's SQS operational readiness? The failure mode of Option B
   (silent billing failures) is only worse than Option A if the DLQ monitoring
   is not in place. Is there a concrete plan to build that monitoring before cutover?

3. What is the payment team's tolerance for publishing to a queue that might
   be backed up? If the queue consumer is behind, payment service has no signal.
   Does the payment team accept that operational model?

4. Are there billing events where eventual consistency is unacceptable?
   If any billing operation has a "must be processed synchronously" requirement
   (e.g., subscription activations that gate service access), Option B requires
   a separate synchronous path for those events.

5. What is the rollback plan if Option B is adopted and a silent failure
   occurs in production before monitoring is fully operational?
```

---

## Step 3: Validate and identify gaps

The engineer reviews the output against known system context:

**Confirmed accurate:**
- The blast radius framing is correct — the triggering incident is accurately represented
- The 500ms requirement caveat is a real concern (consumer throughput has not been benchmarked)
- The dead-letter queue monitoring gap is the most important risk — the team had not fully considered this

**Correction needed:**
- The model assumed no current SQS experience; the billing team has used SQS in a different service but not this one. This changes the operational risk assessment. The engineer notes this.

**What the model couldn't surface:**
- The payment team has already indicated they prefer not to change the interface (they want to keep calling billing synchronously). This is a political/organizational constraint the model has no knowledge of. The engineer adds it to the decision framing.

**New gap surfaced by the analysis:**
- Open question 4 ("are there billing events where eventual consistency is unacceptable?") was not considered. After checking the billing domain, the engineer identifies that subscription activation events must trigger synchronous access grants — these cannot be async without a separate code path.

---

## Step 4: Make and document the decision

The engineer leads a discussion in the architecture review. Decision:

**Adopt a hybrid approach:** Payment events use async queue (Option B) for ledger updates. Subscription activation events retain synchronous processing via a separate endpoint. DLQ monitoring must be implemented and alerting verified in staging before production cutover.

**Conditions:** Decision is contingent on:
1. Load test demonstrating 500ms p99 consumer performance
2. DLQ alerting operational in staging

An ADR is written using the [`adr-template.md`](../../templates/adr-template.md) to record the decision, the hybrid approach rationale, and the two preconditions.

---

## Notes on workflow value

**What the AI analysis added:**
- Open question 4 (subscription activation edge case) was not in the engineer's frame before the analysis. Catching it before implementation avoided building an async architecture that would have required a late redesign.
- The dead-letter queue failure mode was known but not articulated clearly. Having it written out as "this is worse than Option A if monitoring is not in place" made it a concrete pre-condition rather than a vague "we should add monitoring."

**What required engineer judgment:**
- The organizational constraint (payment team prefers synchronous interface) is invisible to the model. The final decision shape depended on it.
- The team's actual SQS experience level was mis-assessed — source context matters.
- The decision to use a hybrid approach (not in the original option set) came from the team, not the model.

**What the model should not have done (and didn't):**
- Recommend an option — output stayed in structured tradeoff format
- Assume the hybrid approach was obvious — it surfaced the tradeoffs and let the team decide
