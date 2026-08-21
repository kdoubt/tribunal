# Decision D1 - worker job-queue claim

Greenfield service on PostgreSQL 16. A `jobs` table has columns
`(id bigserial pk, status text, payload jsonb, worker text, created_at timestamptz)`.
Several identical worker **processes** run this loop to pick up work:

```sql
-- step 1: find the oldest pending job
SELECT id FROM jobs WHERE status = 'pending' ORDER BY created_at LIMIT 1;
-- step 2: (app takes the id, then issues)
UPDATE jobs SET status = 'running', worker = $me WHERE id = $id;
```

The workers run concurrently against the same database.

**Question.** Under concurrent workers, is this scheme safe against **the same
job being processed by two workers**? If it is not, give the fix. Commit to a
call - "safe" or "not safe" - and name the single biggest risk.
