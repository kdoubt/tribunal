The two-step SELECT-then-UPDATE is a TOCTOU race. Both workers can read the same pending `id` before either claim lands, and the UPDATE does not re-check `status`, so both proceed.

CLAIM: Two concurrent workers can both observe the same pending row and both treat it as theirs.
EVIDENCE: Step 1 is an unlocked `SELECT … WHERE status = 'pending' ORDER BY created_at LIMIT 1`. Under PostgreSQL’s default READ COMMITTED, that read takes no exclusive row lock, so N workers can return the same `id` before any UPDATE commits.
CONFIDENCE: 0.97
FALSIFIER: Show that every worker’s SELECT is issued inside a transaction that already holds `FOR UPDATE` (or equivalent exclusive lock) on that row, or that the table has a constraint/trigger that rejects a second claim.

CLAIM: The claim UPDATE cannot fail for the loser, so the race is not self-healing.
EVIDENCE: Step 2 is `UPDATE jobs SET status = 'running', worker = $me WHERE id = $id` with no `AND status = 'pending'` (and no `version`/claim token). The second UPDATE waits, then overwrites `worker` and still reports success; both processes then run the payload.
CONFIDENCE: 0.96
FALSIFIER: Produce the actual statement with a predicate that yields `0` rows for a late claim, plus application code that aborts when `ROW_COUNT = 0`.

CLAIM: Nothing in the given schema serializes or uniquely assigns a running job.
EVIDENCE: Columns are `(id, status, payload, worker, created_at)` only — no unique partial index such as `UNIQUE (id) WHERE status = 'running'`, no claim token, no advisory lock, no `SKIP LOCKED`. `worker` is unconstrained text.
CONFIDENCE: 0.93
FALSIFIER: A unique/exclusion constraint, trigger, or `pg_advisory_lock(id)` around the claim that makes a second `status='running'` write fail.

CLAIM: The standard PostgreSQL fix is a single atomic claim, not a tighter isolation level on the existing two statements.
EVIDENCE: Idiom: `UPDATE jobs SET status='running', worker=$me WHERE id = (SELECT id FROM jobs WHERE status='pending' ORDER BY created_at FOR UPDATE SKIP LOCKED LIMIT 1) RETURNING *`. Equivalent: one `UPDATE … WHERE status='pending' … RETURNING` plus retry on empty. Isolation upgrades (RR/SERIALIZABLE) at best throw serialization errors; they do not make the current unlocked SELECT+unconditional UPDATE correct.
CONFIDENCE: 0.90
FALSIFIER: A documented default-isolation execution of the given two statements where two workers never both receive the same `id` and both see a successful UPDATE.

VERDICT: **not safe** — biggest risk is duplicate processing of one job (double side-effect) because the SELECT is unlocked and the UPDATE does not re-check `pending`.
