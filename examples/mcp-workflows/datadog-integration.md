# Example: DataDog MCP — alert investigation workflow

> **Workflow:** [DataDog Investigation](../../workflows/operations/datadog-investigation.md)
> **Prompt:** [DataDog investigation prompt](../../prompts/operations/datadog-investigation.md)
> **Tool:** Claude Code with DataDog MCP
> **Context:** Representative example of alert investigation using DataDog MCP. Service names and metric values generalized.

---

## What changes with DataDog MCP

Without MCP, the engineer runs DataDog queries manually, copies the results, and pastes them into the AI session. With DataDog MCP, the model can:

- Execute log queries and metric queries directly within the session
- Retrieve metric data for a specified time window without manual copy-paste
- Iterate on queries within the session to refine the investigation
- Pull related signals across services without the engineer context-switching between tools

The workflow and validation steps are identical. MCP compresses the data-gathering loop significantly for multi-service investigations where multiple rounds of querying are needed.

---

## Session transcript (representative)

**Engineer prompt:**

```
Investigate alert: "p99 latency > 2000ms for 5 minutes on [api-service] /v1/orders endpoint"
Alert window: 2026-05-13 09:14 UTC to 09:22 UTC (ongoing as of session start)

Service topology:
- [api-service] calls [order-service] for order creation and retrieval
- [order-service] has a PostgreSQL backend
- [api-service] also calls [inventory-service] for stock checks (sync, in the order creation path)

Deployment context: [order-service] had a deploy at 09:10 UTC (migration: added index on orders.customer_id)
```

**MCP actions taken by Claude Code:**

```
Query 1: [api-service] p99 latency by endpoint, 09:00–09:25 UTC
  → /v1/orders p99: 180ms → 2400ms starting 09:14 UTC
  → Other endpoints: nominal throughout

Query 2: [order-service] response time, 09:00–09:25 UTC  
  → p99: 90ms → 2100ms starting 09:14 UTC
  → Confirms [api-service] latency originates in [order-service]

Query 3: [inventory-service] response time, 09:00–09:25 UTC
  → Nominal (p99 <50ms throughout)
  → Rules out [inventory-service] as a contributor

Query 4: [order-service] database query latency, 09:00–09:25 UTC
  → Slow queries: 0/min before 09:14, 45/min at peak
  → Top slow query: SELECT * FROM orders WHERE customer_id = ? (avg 1800ms at peak)

Query 5: [order-service] logs, 09:10–09:25 UTC, severity=warn or error
  → 09:14 UTC: "index creation in progress — query plans may be suboptimal"
  → 09:14 UTC: table lock acquired on orders table
```

**Model output:**

```
## Signal summary

[api-service] p99 latency degraded starting 09:14 UTC on /v1/orders only. Source traces to 
[order-service], which shows matching latency degradation. [inventory-service] is not 
involved. [order-service] database shows slow queries on `orders WHERE customer_id = ?` 
starting at 09:14 UTC, concurrent with a table lock logged in the service.

Onset at 09:14 UTC is 4 minutes after the [order-service] deployment at 09:10 UTC.

## Candidate root causes

**Candidate 1 (High): Database migration — index creation blocking queries**
- The 09:10 UTC deployment included a migration adding an index on `orders.customer_id`
- Logs at 09:14 UTC confirm: "index creation in progress" and a table lock acquired on orders
- The slow queries are on `orders WHERE customer_id = ?` — exactly the column being indexed
- This is consistent with PostgreSQL's default `CREATE INDEX` behavior acquiring an AccessShareLock 
  that blocks concurrent writes, and potentially with `CREATE INDEX` (non-concurrent) blocking reads 
  depending on PostgreSQL version and isolation level
- Confirmation query: Check [order-service] database connection pool metrics — are connections 
  queuing on the lock? Also check PostgreSQL pg_locks view if accessible.

**Candidate 2 (Low): Deployment itself introduced a regression**
- Less likely given the 4-minute lag between deploy (09:10) and degradation start (09:14)
- The slow query pattern points to the database, not application code
- Confirmation: Check if the degradation correlates precisely with migration completion 
  (index creation finishing) — if latency improves when the index finishes, this is ruled out

## Suggested next queries

1. [order-service] DB connection pool: connections in use / waiting, 09:10–09:25 UTC
2. [order-service] logs: grep for "index" or "lock" from 09:10–09:15 UTC for full migration log
3. Check PostgreSQL version in [order-service] config — determines whether CREATE INDEX 
   was run with CONCURRENTLY option (non-blocking) or without it (blocking)
```

---

## Engineer validation

**Candidate 1 confirmed:** Engineer ran the suggested connection pool query via MCP. Connections waiting spiked to 18/20 at 09:14 UTC, confirming lock contention. Engineer also retrieved the PostgreSQL version: 14.2. The migration ran `CREATE INDEX` without `CONCURRENTLY`, which in PostgreSQL 14 acquires a `ShareLock` that blocks concurrent writes for the duration of index creation.

**Resolution:** The index creation completed at 09:21 UTC (7 minutes). Latency normalized immediately after. No user data impact — orders were queued, not dropped.

**Action item created:** Require `CREATE INDEX CONCURRENTLY` in all future PostgreSQL migrations. Added to the [service]'s migration guidelines and flagged for review of the existing migration tooling.

---

## Notes on MCP value in this session

Without MCP, this investigation would have required:
1. Engineer manually running 5+ DataDog queries
2. Copying each result into the session
3. Re-running refined queries after the initial analysis

With MCP, the model iterated through the query sequence autonomously in the session, narrowing from the API layer to the database in a single pass. The key insight — that the slow queries were specifically on `customer_id` — came from the model querying slow query metrics directly rather than the engineer pre-selecting which queries to copy.

Total investigation time: ~12 minutes. Estimated without MCP: ~30 minutes (same analysis with manual query-copy-analyze loops).
