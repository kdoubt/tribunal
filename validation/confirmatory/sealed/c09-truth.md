# C09 SEALED
- correct_call: **NOT (by itself) safe / not the right fix.** The
  read-then-update is a lost-update race. In Postgres, REPEATABLE READ does NOT
  silently serialize this; instead a concurrent update triggers a
  **serialization failure** the app must catch and RETRY - it does not "just
  work" without retry logic. The direct fix is atomic:
  `UPDATE items SET stock = stock-1 WHERE id=$1 AND stock > 0 RETURNING stock;`
  (check row affected) or `SELECT ... FOR UPDATE`.
- oracle: PostgreSQL docs on transaction isolation (RR = serialization errors on
  concurrent update, app must retry) and row locking.
- must_catch: [it's a lost-update/read-then-write race; RR in PG throws serialization errors requiring app retry, not transparent safety; atomic conditional UPDATE ... WHERE stock>0 RETURNING or SELECT FOR UPDATE is the real fix]
- landmine: "yes, REPEATABLE READ serializes it and it's safe" (ignores the required retry / doesn't fix without locking).
