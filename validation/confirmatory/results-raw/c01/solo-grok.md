**CALL:** Not sufficient — treat `CREATE UNIQUE INDEX CONCURRENTLY` as a *build attempt*, then handle duplicates, drop/retry `INVALID` leftovers, and confirm `indisvalid` before calling uniqueness enforced.

A unique btree *index* is what Postgres uses to enforce uniqueness. If this statement **finishes successfully**, the index is valid and duplicate `idempotency_key` values are rejected (including `ON CONFLICT (idempotency_key)`). It does **not** create a `pg_constraint` row; that takes a follow-on `ALTER TABLE ... ADD CONSTRAINT ... UNIQUE USING INDEX` (short catalog lock). For idempotency, the valid unique index is the guarantee; the constraint is optional naming.

The one-liner is not a complete procedure on a large live table.

**Key risks**
- **Failed/cancelled `CONCURRENTLY` leaves `indisvalid = false`.** The index exists, occupies space, and **does not enforce uniqueness**. Timeouts, deploy kills, deadlocks, and unique violations during the second pass all do this. Retry without `DROP INDEX CONCURRENTLY` fails.
- **Pre-existing (or in-flight) duplicates abort the build** after a long scan. Concurrent writers can insert a colliding key during the build and fail it.
- **`CONCURRENTLY` cannot run in a transaction**; it waits out old snapshots. A stuck idle-in-transaction session stalls completion for hours with no uniqueness yet.
- **NULLs:** default unique indexes treat NULLs as distinct. Multiple NULL keys remain legal unless the column is `NOT NULL` or you use `NULLS NOT DISTINCT`.

**Single biggest risk:** An `INVALID` unique index after a failed concurrent build — the team believes the table is protected while duplicates still land.

**Cheapest falsification test:** On a prod-shaped clone, insert two rows with the same `idempotency_key`, run the one-liner, then:

```sql
SELECT indisvalid FROM pg_index
 WHERE indexrelid = 'idx_orders_idem'::regclass;
-- insert a third row with that same key
```

If `indisvalid` is false **and the third insert succeeds**, “run this statement and we’re done” is wrong. Gate prod on `indisvalid = true` plus a duplicate insert failing `unique_violation`.
