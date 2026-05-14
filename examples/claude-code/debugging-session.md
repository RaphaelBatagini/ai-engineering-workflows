# Example: Debugging session — connection pool exhaustion

> **Workflow:** [Debugging](../../workflows/engineering/debugging.md)
> **Prompt:** [Debugging prompt](../../prompts/engineering/debugging.md)
> **Tool:** Claude Code
> **Context:** Representative session based on a real debugging investigation. Service names and specific values are generalized.

---

## Situation

A Go microservice handling API requests began returning `503` errors intermittently under normal load. The issue was not present in staging and did not reproduce locally. Error rate was approximately 2% of requests, clustering in 30–60 second windows before resolving.

---

## Step 1: Failure description (engineer-authored)

```
Observed behavior: API endpoints return HTTP 503 at ~2% rate under normal production traffic
Expected behavior: 503s should only occur during extreme load or downstream service outages
Reproduction: Cannot reliably reproduce — intermittent under production traffic
Frequency: ~2% of requests; appears to cluster in 30-60 second windows, then self-resolve
First observed: ~4 hours after most recent deployment (2026-04-15 ~18:00 UTC)
Recent changes: Database client library updated (v2.1.0 → v2.3.0) in the deployment
Environment: Production; 3 replicas; PostgreSQL RDS backend; Go 1.22
```

---

## Step 2: Context assembled by engineer

**Stack trace (exact):**

```
goroutine 847 [running]:
database/sql.(*DB).conn(0xc0001e4000, 0x172f040, 0xc000060660, 0x1, 0x0, 0x0, 0x0)
        /usr/local/go/src/database/sql/sql.go:1283 +0x3a5
database/sql.(*DB).QueryContext(0xc0001e4000, 0x172f040, 0xc000060660, 0xc000254140, 0x1b, 0x0, 0x0)
        /usr/local/go/src/database/sql/sql.go:1619 +0x89
[service]/internal/repository.(*UserRepository).GetByID(...)
        /app/internal/repository/user.go:47 +0x9c
```

**Log excerpts:**

```
2026-04-15T18:23:41Z ERROR request failed: context deadline exceeded waiting for connection
2026-04-15T18:23:41Z ERROR db pool exhausted: 0/25 connections available, 127 waiting
2026-04-15T18:23:42Z ERROR request failed: context deadline exceeded waiting for connection
2026-04-15T18:24:08Z INFO  pool recovered: 18/25 connections available
```

**Source context (engineer reviewed `user.go` after noting the library update):**

```go
func (r *UserRepository) GetByID(ctx context.Context, id int64) (*User, error) {
    rows, err := r.db.QueryContext(ctx, "SELECT * FROM users WHERE id = $1", id)
    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)
    }
    // Note: rows.Close() is absent — engineer noticed this before running the prompt
    var user User
    if rows.Next() {
        if err := rows.Scan(&user.ID, &user.Email, &user.Name); err != nil {
            return nil, err
        }
    }
    return &user, nil
}
```

---

## Step 3: Model output (hypothesis generation)

**Hypothesis 1 (highest likelihood): `rows` not closed — connection not released**

- **Hypothesis:** `rows.QueryContext` returns a `*sql.Rows` object that holds a database connection. Without a `defer rows.Close()` call, the connection is not returned to the pool until the garbage collector finalizes the object. Under concurrent traffic, connections accumulate faster than GC reclaims them, exhausting the pool (25 connections).
- **Supporting evidence:** Logs confirm pool exhaustion: `0/25 connections available, 127 waiting`. Stack trace shows failure at `sql.DB.conn`, consistent with pool exhaustion. The source code shows `QueryContext` called without a subsequent `rows.Close()` or `defer rows.Close()`.
- **Contradicting evidence:** The issue appeared 4 hours after deployment rather than immediately — suggesting gradual accumulation rather than an instant exhaustion. This is consistent with slow connection leaking rather than immediate exhaustion.
- **Confirmation test:** Add `defer rows.Close()` immediately after the `QueryContext` call. Deploy to a canary and observe pool metrics — connection count should stabilize.
- **Elimination test:** If pool metrics show normal connection cycling after the fix, H1 is confirmed. If connections continue leaking, the issue is elsewhere.
- **Priority:** High

**Hypothesis 2: Library v2.3.0 changed connection lifecycle behavior**

- **Hypothesis:** The database client library update changed how connections are held internally (e.g., multiplexing behavior, keepalive settings), causing connections to be held longer with the existing calling code.
- **Supporting evidence:** Issue appeared after the v2.1.0 → v2.3.0 update. 4-hour lag could reflect accumulation under changed library behavior.
- **Contradicting evidence:** The missing `rows.Close()` in H1 is a sufficient and direct explanation. H2 is not needed to explain the data.
- **Confirmation test:** Review the v2.3.0 changelog for connection lifecycle changes.
- **Elimination test:** Pin the library to v2.1.0 in a test environment and reproduce the load pattern. If pool exhaustion disappears, H2 is contributing. If the `rows.Close()` fix is already in place, H2 becomes irrelevant.
- **Priority:** Medium (only relevant if H1 fix doesn't fully resolve the issue)

**Hypothesis 3: Pool size insufficient for actual concurrency**

- **Hypothesis:** The pool size (25 connections) is too small for the actual concurrent request volume, and the 503s are a legitimate capacity limit being hit.
- **Supporting evidence:** `127 waiting` suggests significant queuing. The cluster/burst pattern could indicate traffic spikes.
- **Contradicting evidence:** The issue is new — same pool size with same traffic patterns before the deployment did not produce this behavior.
- **Priority:** Low — unlikely given the deployment correlation, but worth keeping if H1 and H2 don't resolve it

---

## Engineer validation (Step 3 triage)

Engineer applied system knowledge to the model output:

**H1 elevated to confirmed:** The engineer had already noticed the missing `rows.Close()` when assembling source context in Step 2. The model independently identified the same cause. The engineer also confirmed via `git log` that `rows.Close()` was removed during the v2.3.0 migration refactor — it was present in the v2.1.0 code path but removed when the library's new API was adopted without the close call being carried over.

**H2 investigated and eliminated:** Engineer reviewed the v2.3.0 changelog. No connection lifecycle changes. H2 eliminated.

**H3 dismissed:** Same traffic volume, no 503s before the deployment. Pool size is not the root cause.

---

## Step 4: Fix

```go
func (r *UserRepository) GetByID(ctx context.Context, id int64) (*User, error) {
    rows, err := r.db.QueryContext(ctx, "SELECT * FROM users WHERE id = $1", id)
    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)
    }
    defer rows.Close() // restores connection release after query completion

    var user User
    if rows.Next() {
        if err := rows.Scan(&user.ID, &user.Email, &user.Name); err != nil {
            return nil, err
        }
    }
    return &user, nil
}
```

Engineer audited all other repository methods for the same pattern. Two additional `GetBy*` methods had the same issue — fixed in the same PR.

**Regression test added:**

```go
func TestUserRepository_ConnectionNotLeaked(t *testing.T) {
    db := setupTestDB(t)
    repo := NewUserRepository(db)
    
    initialStats := db.Stats()
    for i := 0; i < 50; i++ {
        _, _ = repo.GetByID(context.Background(), int64(i))
    }
    finalStats := db.Stats()
    
    // No connections should be in use after all queries complete
    assert.Equal(t, 0, finalStats.InUse, "connections leaked after GetByID calls")
    assert.Equal(t, initialStats.OpenConnections, finalStats.OpenConnections)
}
```

---

## Notes on workflow value

The model correctly identified the root cause from the source code in Hypothesis 1. The engineer's key contributions were:

1. **Context assembly:** Reviewing `user.go` before running the prompt provided the source code that made the missing `rows.Close()` visible. Without source context, the model would have had only the log/stack evidence — likely still generating H1, but with less confidence.
2. **Codebase audit:** The model identified the bug in `GetByID`. The engineer audited the full repository for the same pattern — finding two additional instances.
3. **Regression test:** The model did not generate the test. The engineer wrote it to prevent re-introduction of the same class of bug.

Hypothesis generation compressed from ~20 minutes (manual analysis) to ~3 minutes. The engineer spent the saved time on the audit and regression coverage.
