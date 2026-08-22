**CLAIM 1**  
**CLAIM:** Not sufficient as-is: concurrent creation can fail on existing or racing duplicate keys.  
**EVIDENCE:** PostgreSQL performs two scans. A uniqueness violation or deadlock aborts the command but leaves an `INVALID` index that consumes write overhead; after second-scan enforcement starts, it may even reject writes despite later build failure. [PostgreSQL 16: CREATE INDEX](https://www.postgresql.org/docs/16/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY)  
**CONFIDENCE:** Very high.  
**FALSIFIER:** Successful completion followed by `pg_index.indisvalid = true`.

**CLAIM 2**  
**CLAIM:** A valid unique index enforces uniqueness, but it is not a declarative `UNIQUE` constraint recorded in `pg_constraint`.  
**EVIDENCE:** PostgreSQL enforces uniqueness through unique indexes, but converting it into a schema constraint requires:  
```sql
ALTER TABLE orders
  ADD CONSTRAINT orders_idempotency_key_key
  UNIQUE USING INDEX idx_orders_idem;
```  
This attachment is normally fast, though it requires a brief `ACCESS EXCLUSIVE` lock and should use a short `lock_timeout`. [PostgreSQL 16: ALTER TABLE](https://www.postgresql.org/docs/16/sql-altertable.html)  
**CONFIDENCE:** High.  
**FALSIFIER:** A `UNIQUE` row for this index already exists in `pg_constraint`.

**CLAIM 3**  
**CLAIM:** Default uniqueness permits multiple `NULL` keys.  
**EVIDENCE:** Unique indexes default to `NULLS DISTINCT`. Use `NULLS NOT DISTINCT` or separately enforce `NOT NULL` if null must not bypass idempotency. [PostgreSQL 16: unique constraints](https://www.postgresql.org/docs/16/ddl-constraints.html#DDL-CONSTRAINTS-UNIQUE-CONSTRAINTS)  
**CONFIDENCE:** Very high.  
**FALSIFIER:** The column is already `NOT NULL`, or multiple nulls are explicitly acceptable.

**CLAIM 4**  
**CLAIM:** “No write downtime” does not mean operationally harmless: the build waits on transactions, adds CPU/I/O and write amplification, cannot run inside a transaction block, and is unsupported directly on a partitioned parent.  
**EVIDENCE:** Documented concurrent-build caveats.  
**CONFIDENCE:** Very high.  
**FALSIFIER:** None affecting correctness; workload testing can show negligible operational impact.

**VERDICT:** **Not sufficient — audit/remediate duplicates, decide NULL semantics, run concurrently, verify `indisvalid`, clean up/retry failures, then optionally attach as a named constraint.** Biggest risk: duplicate-driven failure leaving an invalid index with surprising write overhead/enforcement. Cheapest discriminating test: run `SELECT idempotency_key FROM orders GROUP BY 1 HAVING count(*) > 1 LIMIT 1;` on a recent replica/snapshot, then verify the production build’s catalog state.
