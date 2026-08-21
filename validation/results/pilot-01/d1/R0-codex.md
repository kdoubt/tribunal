CLAIM: Not safe.  
EVIDENCE: This is a time-of-check/time-of-use race: multiple workers can complete the `SELECT` before either performs the `UPDATE`, receive the same `id`, and each proceed as though it claimed the job.  
CONFIDENCE: 1.0  
FALSIFIER: Proof that the two statements always execute under serialization or retain an exclusive row lock between selection and update.

CLAIM: Claim jobs atomically using row locking and skip already-locked candidates.  
EVIDENCE: PostgreSQL’s `FOR UPDATE SKIP LOCKED` is designed for queue-like consumers. In one transaction:

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
WHERE id = $id
RETURNING *;

COMMIT;
```

Only process a job returned by the update.  
CONFIDENCE: 0.99  
FALSIFIER: Workers process the selected ID even when locking/updating fails, or the transaction releases the lock before recording the claim.

CLAIM: A single-statement claim is preferable because it removes the application-level gap.  
EVIDENCE:

```sql
UPDATE jobs
SET status = 'running', worker = $me
WHERE id = (
  SELECT id
  FROM jobs
  WHERE status = 'pending'
  ORDER BY created_at, id
  FOR UPDATE SKIP LOCKED
  LIMIT 1
)
RETURNING *;
```

Each worker processes only the returned row.  
CONFIDENCE: 0.99  
FALSIFIER: PostgreSQL does not preserve the subquery’s row lock through completion of the containing statement.

VERDICT: **NOT SAFE — use an atomic `FOR UPDATE SKIP LOCKED` claim with `RETURNING`; the biggest risk is duplicate processing and duplicated external side effects.**
