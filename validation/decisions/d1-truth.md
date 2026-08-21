# D1 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (discriminator candidate)
- **correct_call:** **NOT safe.** The `SELECT` and the `UPDATE` are two separate
  statements; under READ COMMITTED (the default), two workers can both `SELECT`
  the same oldest `pending` id before either runs its `UPDATE`, so both process
  that job. This is a classic read-then-write race.
- **oracle:** PostgreSQL docs, "Row Locking" / `SELECT ... FOR UPDATE SKIP
  LOCKED` (§ Explicit Locking 13.3.2) - SKIP LOCKED is documented specifically
  to "avoid lock contention with multiple consumers accessing a queue-like
  table."
- **correct fix (any one):**
  1. Atomic claim in one statement:
     `UPDATE jobs SET status='running', worker=$me WHERE id = (SELECT id FROM jobs WHERE status='pending' ORDER BY created_at FOR UPDATE SKIP LOCKED LIMIT 1) RETURNING id;`
  2. `SELECT ... FOR UPDATE SKIP LOCKED LIMIT 1` inside a transaction, then
     UPDATE, then COMMIT.
- **must_catch:**
  1. the race exists *between* the SELECT and the UPDATE (two workers read the same id);
  2. adding an **index** does not fix it (it is a concurrency bug, not a speed bug);
  3. wrapping the two statements in a plain transaction (READ COMMITTED) alone does **not** fix it without row locking;
  4. `FOR UPDATE SKIP LOCKED` (or an atomic `UPDATE ... WHERE id=(subselect ... FOR UPDATE SKIP LOCKED) RETURNING`) is the mechanism.
- **landmine (confident wrong answers):** "safe - `LIMIT 1` makes it atomic";
  "just add an index on `(status, created_at)`"; "wrap it in a transaction and
  it's fine" (without row locking / SKIP LOCKED).
