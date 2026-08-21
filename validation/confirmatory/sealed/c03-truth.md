# C03 SEALED
- correct_call: **NOT correct.** If the subquery returns even one NULL
  `customer_id`, `NOT IN` evaluates to NULL (not TRUE) for every outer row, so the
  query returns **zero rows**. Classic three-valued-logic trap.
- oracle: SQL standard 3VL; Postgres docs on NOT IN / NULL.
- fix: use `NOT EXISTS` (correlated) or add `WHERE customer_id IS NOT NULL` to the
  subquery, or `LEFT JOIN ... WHERE x IS NULL` (anti-join).
- must_catch: [a NULL in the subquery makes NOT IN return no rows; NOT EXISTS/anti-join is the fix; NULL != empty result-set semantics]
- landmine: "correct" / "just add an index" / not noticing the nullable column.
