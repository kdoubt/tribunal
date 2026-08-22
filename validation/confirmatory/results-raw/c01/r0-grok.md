**CALL: not sufficient — handle duplicate cleanup + INVALID-index retry (then optionally attach the constraint).**

**CLAIM 1.** `CREATE UNIQUE INDEX CONCURRENTLY` is the right *core* DDL (it avoids an ACCESS EXCLUSIVE rewrite), but a failed concurrent build does **not** roll back cleanly: Postgres leaves an `INVALID` index that **does not enforce uniqueness**.
**EVIDENCE:** Concurrent unique builds run two table scans outside a transaction. Uniqueness violations (or other errors) abort the command and still catalog the index with `pg_index.indisvalid = false`. Writes keep succeeding; the team can believe the guarantee is live.
**CONFIDENCE:** 95
**FALSIFIER:** After a forced duplicate-induced failure, `indisvalid` is true and a second identical `INSERT` is rejected.

**CLAIM 2.** On a table that lived without this guarantee, duplicate `idempotency_key` values (historical or in-flight during the second scan) are expected; the one-liner then fails until they are merged/deleted, then `DROP INDEX CONCURRENTLY` + retry.
**EVIDENCE:** Unique concurrent builds check existing rows *and* rows inserted after the first snapshot. Any duplicate key fails the build. Retry without drop hits “already exists” on the invalid index.
**CONFIDENCE:** 90
**FALSIFIER:** `SELECT idempotency_key FROM orders GROUP BY 1 HAVING COUNT(*) > 1` is empty for the whole table *and* stays empty across a dual-insert race during the build.

**CLAIM 3.** A unique **index** is not a unique **constraint**. Enforcement is the same for inserts; catalog/FK/`DROP CONSTRAINT`/dump shape are not.
**EVIDENCE:** FKs and `pg_constraint` require a constraint. The non-locking promotion is `ALTER TABLE ... ADD CONSTRAINT ... UNIQUE USING INDEX idx_orders_idem` (SHARE UPDATE EXCLUSIVE — writes continue).
**CONFIDENCE:** 88
**FALSIFIER:** After the one-liner, `pg_constraint` already contains a unique constraint on `orders.idempotency_key`.

**CLAIM 4.** Default UNIQUE treats NULLs as distinct (unless `NULLS NOT DISTINCT` / a partial index). Nullable keys will still allow many NULL rows.
**EVIDENCE:** Postgres unique-index NULL semantics; PG15+ `NULLS NOT DISTINCT` is opt-in.
**CONFIDENCE:** 85
**FALSIFIER:** Column is `NOT NULL`, or product spec explicitly allows duplicate NULLs.

**VERDICT:** **Not sufficient — handle X = pre-existing/in-flight duplicates + drop/retry of leftover INVALID indexes (then `ADD CONSTRAINT ... USING INDEX` if you actually need a constraint).** Single biggest risk: failed concurrent build leaves an INVALID index and uniqueness is **not** enforced while the team thinks it is. Cheapest discriminating test: count duplicate keys (`GROUP BY idempotency_key HAVING COUNT(*) > 1`); after any run, assert `pg_index.indisvalid` (and `pg_constraint` if a constraint was the goal).
