## Decision Memo — C09

**CALL:** **Yes—on PostgreSQL, REPEATABLE READ prevents this same-row oversell, but do not ship the isolation-only change; use one conditional `UPDATE` under READ COMMITTED.**

At REPEATABLE READ, both transactions may initially read `stock = 1`, but after one commits its update, PostgreSQL aborts the competing updater with SQLSTATE `40001` (“could not serialize access due to concurrent update”). Thus, both decrements cannot commit. PostgreSQL requires the application to retry the **entire transaction**. [PostgreSQL transaction-isolation documentation](https://www.postgresql.org/docs/current/transaction-iso.html)

Preferred implementation:

```sql
UPDATE items
SET stock = stock - 1
WHERE id = $1
  AND stock > 0
RETURNING stock;
```

Treat one returned row as success and zero rows as sold out. This is atomic and safe at READ COMMITTED. Add `CHECK (stock >= 0)` as defense in depth.

**Single biggest risk:** The application does not correctly handle `40001` transaction aborts—especially if it records payment, sends confirmation, or reports checkout success before the database commit succeeds. The result may not be negative stock, but it can be charged customers with no committed reservation or elevated checkout failures under contention. PostgreSQL does not retry automatically. [Serialization-failure handling](https://www.postgresql.org/docs/current/mvcc-serialization-failure-handling.html)

Other risks:

- Hot products generate repeated aborts and retries.
- Retrying only the failed statement, rather than the whole transaction, is incorrect.
- The guarantee does not cover inventory represented across multiple rows or external systems.

**Cheapest falsification test:** Initialize stock to `1`; synchronize 100 concurrent transactions so all complete the read before any update. Assert:

1. exactly one transaction commits a purchase;
2. final stock is `0`;
3. every other transaction receives/handles `40001` or returns sold-out after a full retry;
4. exactly one payment/order side effect exists.

Any second committed purchase means the design is wrong.
