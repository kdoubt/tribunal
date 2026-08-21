**CALL: No. REPEATABLE READ does not make this oversell-safe. Keep READ COMMITTED and decrement atomically: `UPDATE items SET stock = stock - 1 WHERE id = $1 AND stock > 0`, require `rowcount = 1`.**

Raising isolation leaves a TOCTOU: a non-locking `SELECT`, an app-side `stock > 0` check, then an unconditional `UPDATE`. Isolation does not attach the business predicate to the write, and it does nothing if the two statements are not one transaction (autocommit, ORM session, two round-trips).

**Key risks of shipping RR**
- **MySQL/InnoDB:** `UPDATE` is a current read. Both sessions snapshot `stock = 1`, both pass the app check, both decrement; stock goes negative.
- **Postgres:** second writer dies with `40001` instead of enforcing `stock > 0`. That is an abort, not a correct checkout.
- **Retry handlers** (the usual “retry on serialization failure”) re-issue only the `UPDATE` and decrement through zero.
- Extra aborts, latency, and ops noise with no `CHECK (stock >= 0)` in the database.

**Single biggest risk:** The team treats the isolation bump as the fix, then a serialization retry (or a current-read `UPDATE`) still commits extra sales. Inventory and orders diverge in production; RR looks “on” in the session while the race remains.

**Cheapest disproof:** Seed `stock = 1`. Fire two concurrent checkouts of that SKU. Fail if `completed_orders > 1` or `items.stock < 0`. That test is ~10 lines and kills the RR change immediately; the conditional `UPDATE` (or `SELECT … FOR UPDATE` in one txn) is what makes it pass.
