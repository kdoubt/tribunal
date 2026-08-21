# Decision C01 - add a uniqueness constraint to a large live table

A team must add a uniqueness guarantee on `orders(idempotency_key)` for an
existing **large, high-write Postgres 16 table** in production, with **no write
downtime**. The proposed one-liner:

```sql
CREATE UNIQUE INDEX CONCURRENTLY idx_orders_idem ON orders (idempotency_key);
```

**Question.** Is running this statement, on its own, a safe and *sufficient* way
to end up with an enforced uniqueness constraint - or are there failure modes the
team must handle for this to be correct? Commit to a call ("sufficient as-is" or
"not sufficient - handle X") and name the single biggest risk.
