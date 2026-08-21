I'll ground the uniqueness call in the exact MAX-then-INSERT path and whether anything in the schema actually serializes or rejects collisions.The repo has a D5 brief and a seat template. I'll follow those, and I will not open the truth file.**Framings considered:** (1) “reliably unique under concurrent web requests as written”; (2) “unique enough if traffic is low”; (3) “unique if we quietly wrap it in a transaction.” The brief asks (1). Low traffic is not a uniqueness guarantee. A transaction is not specified, and default isolation would not save this pattern anyway.

---

**CLAIM 1.** Concurrent invoice creates can compute the same next `invoice_no` because allocation is a two-statement TOCTOU on `MAX`.

**EVIDENCE.** Brief (`d5-brief.md:8-11`): `SELECT COALESCE(MAX(invoice_no), 0) + 1` then a later `INSERT … VALUES (…, $next)`. Two requests can both read `MAX = N` before either insert commits, so both write `N+1`. `EXTERNAL`: PostgreSQL *Transaction Isolation* — READ COMMITTED (default) lets a statement see only rows committed before it started; T2’s `MAX` does not see T1’s uncommitted insert, and there is no predicate lock on the aggregate.

**CONFIDENCE.** 0.95

**FALSIFIER.** A single-statement atomic allocator with exclusive locking (or app-wide serialization of invoice creates) such that a second `SELECT MAX` cannot run until the first `INSERT` is committed.

---

**CLAIM 2.** Collisions are committed, not rejected: there is no uniqueness enforcement on `invoice_no`.

**EVIDENCE.** Brief (`d5-brief.md:13`): “There is currently no unique constraint on `invoice_no`.” Without `UNIQUE`/`PRIMARY KEY` on that column, two `INSERT`s of the same `invoice_no` are both legal.

**CONFIDENCE.** 0.98

**FALSIFIER.** A unique index/constraint on `invoice_no` already exists in the live schema and the brief is wrong.

---

**CLAIM 3.** Putting the two statements in a default transaction does not make numbers unique.

**EVIDENCE.** `EXTERNAL`: PostgreSQL *Transaction Isolation* — READ COMMITTED does not serialize `MAX()`; both transactions can read the same max and both commit. REPEATABLE READ still allows this write skew on an aggregate. SERIALIZABLE SSI *may* abort one writer, but the brief specifies neither isolation, `SELECT FOR UPDATE`, nor retry.

**CONFIDENCE.** 0.90

**FALSIFIER.** Invoice creates actually run at `SERIALIZABLE` (or hold an exclusive lock / advisory lock covering the allocator) **and** retry on serialization failure, so a colliding attempt never commits.

---

**CLAIM 4.** Uniqueness needs an atomic allocator plus a uniqueness constraint as the safety net; `MAX+1` in the app is neither.

**EVIDENCE.** `EXTERNAL`: PostgreSQL `CREATE SEQUENCE` / identity columns issue distinct values under concurrency; `SELECT … FOR UPDATE` on a one-row counter serializes allocation; `UNIQUE (invoice_no)` turns any leftover race into an error instead of a silent duplicate. A sequence can leave gaps; a locked counter is the usual way to get unique *and* gapless-ish.

**CONFIDENCE.** 0.88

**FALSIFIER.** A measured concurrent create test (two+ overlapping inserts) that never produces duplicate `invoice_no` values under the proposed SQL, with no extra lock, unique constraint, or retry.

---

**VERDICT: NO** — concurrent `MAX+1` then insert, with no unique constraint, will emit duplicate invoice numbers. Biggest risk: silent duplicates in production (two invoices share `INV-N`); fix with `UNIQUE (invoice_no)` immediately and replace app-side `MAX+1` with a locked counter (`SELECT FOR UPDATE`) or a sequence plus conflict retry.
