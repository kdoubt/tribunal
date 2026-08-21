## Decision memo — Choose B: CRUD + in-database append-only audit history

**Recommendation:** Keep a conventional mutable Postgres `orders` model and add an
append-only `order_history` table written **in the database** (an `AFTER INSERT
OR UPDATE OR DELETE` row trigger capturing the full old/new row as JSONB in the
same transaction), not in application code. Store actor, timestamp, operation
type, reason/correlation id, and full-row snapshots per version.

**Why B, at this scale and for this team:**

- **The scale does not select event sourcing.** Thousands of orders/day (even
  ~10k orders × ~20 mutations ≈ 2×10⁵ history rows/day) is routine for one
  Postgres primary. Event sourcing's performance wins are against CRUD lock
  contention that does not arise here.
- **Audit + "order as of T" don't require the event log to be the system of
  record.** A transactional full-row history plus snapshots answers both with a
  simple `(order_id, changed_at)` index. Fowler (*Event Sourcing*, 2005) notes a
  change log is "a small gain" obtainable by keeping history or a log — the
  distinctive ES facilities are full rebuild and *replay of corrections*, which
  aren't asked for. Microsoft's temporal-table guidance prescribes
  system-versioned tables precisely for audit and point-in-time reconstruction.
- **ES imposes correctness machinery unrelated to the stated needs** — event
  schema evolution, deterministic replay, projection checkpoints/rebuilds,
  idempotent consumers, version/concurrency handling, eventual consistency — for
  a six-person Postgres/REST team with **no ES experience**. Microsoft's
  Event-Sourcing pattern itself says it "might not be suitable" when data "works
  well with traditional CRUD" or "consistency and real-time updates to the views
  are required," and that changing persisted event format "can be difficult."
- **ES doesn't automatically make historical truth easy:** replay under evolved
  code/projection logic can diverge from original behavior; snapshots record the
  materialized state directly.

**Single biggest risk:** history implemented in **application code** rather than
in-DB, so some write paths — a missed handler, admin SQL, an UPDATE that commits
before the audit insert — never record, and "as of T" is *silently* wrong while
everything looks healthy. Mitigate by enforcing capture in the database (row
trigger) and revoking `UPDATE`/`DELETE` on `order_history`.

**Cheapest test that says you chose wrong:** a two-day spike with trigger-written
full-row JSONB history; run the real support/finance "as of T" questions, and
list credible near-term consumers. Switch to A **only** if a required question
needs *intent* or *replay-with-correction* that a snapshot can't answer, or ≥2
committed consumers need independent replay/projections from an authoritative
order-event stream. If the spike answers the questions, A was the overcommit.
