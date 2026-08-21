**CLAIM:** Switching this session to REPEATABLE READ does **not** make the check-then-act checkout oversell-safe.
**EVIDENCE:** The app still decides from a `SELECT` and then writes `stock = stock - 1` with no `AND stock > 0`. RR does not lock the predicate `stock > 0` and does not make that pair atomic. In InnoDB RR, `SELECT` is a snapshot read while `UPDATE` is a current read: both txns can see `stock = 1`, both pass, the second `UPDATE` applies to 0 and commits `-1`.
**CONFIDENCE:** 0.9
**FALSIFIER:** Under RR, two concurrent txns against `stock = 1` yield exactly one committed decrement and `stock >= 0` with no extra app locking — on MySQL as well as Postgres.

**CLAIM:** Postgres RR only *sometimes* papers over this, via abort, not via a correct reserve.
**EVIDENCE:** PG RR is snapshot isolation. If (and only if) both statements share one txn, the loser’s `UPDATE` hits `could not serialize access due to concurrent update` (40001). That is not oversell-safe unless 40001 is treated as failure and the **entire** SELECT→check→UPDATE is retried. Autocommit, catching/ignoring 40001, or retrying only the `UPDATE` still oversells.
**CONFIDENCE:** 0.85
**FALSIFIER:** Same race, one txn, no retry logic: two sales never commit and `stock` never goes negative even if 40001 is swallowed.

**CLAIM:** The cheap correct fix is a single conditional update (or `SELECT … FOR UPDATE` in one txn).
**EVIDENCE:** `UPDATE items SET stock = stock - 1 WHERE id = $1 AND stock > 0` then check `rowcount = 1`. One round-trip, works in READ COMMITTED, needs no isolation bump. A `CHECK (stock >= 0)` constraint is the backstop the isolation change still omits.
**CONFIDENCE:** 0.95
**FALSIFIER:** That `UPDATE` allowing two concurrent successes when starting `stock = 1`.

**CLAIM:** Contention abort is the operational cost if you still “just raise RR.”
**EVIDENCE:** Hot SKU → serialization failures / deadlocks under load; without a bounded retry-from-SELECT you convert oversell into checkout outages.
**CONFIDENCE:** 0.8
**FALSIFIER:** Last-item stampede under PG RR with no 40001 and no extra latency vs RC + conditional `UPDATE`.

**VERDICT:** **NO — do not ship RR as the oversell fix.** Biggest risk: last unit sold twice (`stock` negative or two orders). Cheapest test: two concurrent checkouts on `stock = 1`; assert one success, `stock = 0`, no 40001-as-success.
