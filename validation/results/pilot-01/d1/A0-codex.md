**CALL: Not safe.**

**Reasoning:** The `SELECT` and `UPDATE` are separate operations. Concurrent workers can select the same oldest pending job before either updates it. Each can then update it and process it, with the last update merely overwriting `worker`.

**Single biggest risk:** Duplicate processing of the same job.

**Fix:** Atomically claim one row inside a transaction using a row lock:

```sql
BEGIN;

SELECT id
FROM jobs
WHERE status = 'pending'
ORDER BY created_at, id
FOR UPDATE SKIP LOCKED
LIMIT 1;

UPDATE jobs
SET status = 'running', worker = $me
WHERE id = $id;

COMMIT;
```

Prefer one statement:

```sql
WITH claimed AS (
  SELECT id
  FROM jobs
  WHERE status = 'pending'
  ORDER BY created_at, id
  FOR UPDATE SKIP LOCKED
  LIMIT 1
)
UPDATE jobs AS j
SET status = 'running', worker = $me
FROM claimed
WHERE j.id = claimed.id
RETURNING j.*;
```

Run that statement in a transaction and process only the returned job. `SKIP LOCKED` lets other workers claim different jobs instead of waiting.
