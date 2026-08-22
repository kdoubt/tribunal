## Decision memo C01

**CALL:** **Not sufficient—handle build failure/invalid-index cleanup, verify NULL semantics, and attach the valid index as a declared constraint.**

A successful, valid unique index does enforce uniqueness against inserts and updates. However, this one-liner is not a reliable migration by itself:

- Existing or concurrent duplicate keys can make the concurrent build fail. PostgreSQL then leaves an `INVALID` index behind; it consumes write overhead and may continue enforcing uniqueness for new writes despite being incomplete and unusable for queries. Clean it up and retry after resolving duplicates. [PostgreSQL 16: CREATE INDEX](https://www.postgresql.org/docs/16/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY)
- `NULL` values are distinct by default, so multiple `NULL` idempotency keys remain legal. Use `NULLS NOT DISTINCT` or `NOT NULL` if that violates the intended guarantee.
- A unique index is enforcement machinery, not a declared `pg_constraint`. After confirming validity, install it as metadata without rebuilding:

```sql
ALTER TABLE orders
  ADD CONSTRAINT orders_idempotency_key_key
  UNIQUE USING INDEX idx_orders_idem;
```

This metadata step requires a brief `ACCESS EXCLUSIVE` lock, so use a short `lock_timeout` and retry rather than waiting behind traffic. PostgreSQL explicitly recommends the concurrent-index-then-attach workflow. [PostgreSQL 16: ALTER TABLE](https://www.postgresql.org/docs/16/sql-altertable.html)

**Single biggest risk:** treating a failed concurrent build as harmless or complete when it has left an invalid, incomplete index that still adds write cost—and may reject some writes—without guaranteeing all existing data is unique.

**Cheapest decisive criterion:** require the statement to succeed and this postcondition to return one valid index plus one constraint:

```sql
SELECT i.indisvalid, i.indisready, c.conname
FROM pg_index i
JOIN pg_class x ON x.oid = i.indexrelid
LEFT JOIN pg_constraint c ON c.conindid = i.indexrelid
WHERE x.oid = 'idx_orders_idem'::regclass;
```

Any false validity flag or missing constraint means the rollout is incomplete.
