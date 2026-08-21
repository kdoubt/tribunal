**CALL: not safe.**

The plain `SELECT` takes no row lock. Under PostgreSQL’s default `READ COMMITTED` isolation, concurrent workers can read the same committed `pending` row from their respective statement snapshots. Both then update that same `id`; the second update may wait, but because its predicate is only `WHERE id = $id`, it still matches after the first update and overwrites `worker`. Both applications may consequently process the job. [PostgreSQL 16 transaction-isolation semantics](https://www.postgresql.org/docs/16/transaction-iso.html)

**Biggest risk:** duplicate job execution, potentially duplicating irreversible side effects.

**Fix:** atomically claim one locked row, using `SKIP LOCKED` for concurrent queue consumers:

```sql
WITH next_job AS (
    SELECT id
    FROM jobs
    WHERE status = 'pending'
    ORDER BY created_at, id
    FOR UPDATE SKIP LOCKED
    LIMIT 1
)
UPDATE jobs AS j
SET status = 'running',
    worker = $me
FROM next_job
WHERE j.id = next_job.id
RETURNING j.id, j.payload;
```

Process a job only when this statement returns a row. `FOR UPDATE` locks the selected row, while `SKIP LOCKED` directs other workers to another available job; PostgreSQL explicitly documents this as suitable for multiple consumers of a queue-like table. [PostgreSQL 16 `SELECT` locking clause](https://www.postgresql.org/docs/16/sql-select.html)
