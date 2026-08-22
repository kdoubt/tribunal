CLAIM: The query is **not correct** when the subquery can return `NULL`.  
EVIDENCE: SQL uses three-valued logic. If the result set contains `NULL`, `c.id NOT IN (...)` becomes `UNKNOWN` for every non-matching customer, so the `WHERE` clause rejects them.  
CONFIDENCE: Very high.  
FALSIFIER: Insert a blocked guest order with `customer_id = NULL`; if otherwise eligible customers still appear, the claim is false.

CLAIM: Use a correlated `NOT EXISTS`, which safely ignores unrelated `NULL` customer IDs.

```sql
SELECT *
FROM customers c
WHERE NOT EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.customer_id = c.id
    AND o.status = 'blocked'
);
```

EVIDENCE: A blocked row excludes a customer only when `o.customer_id = c.id` is true; comparison against `NULL` is not true and therefore cannot create a false exclusion.  
CONFIDENCE: Very high.  
FALSIFIER: Any dataset where this returns a customer having a blocked order with the same `customer_id`.

VERDICT: **CALL: not correct—replace `NOT IN` with correlated `NOT EXISTS`.** Biggest risk: one blocked guest order with a `NULL customer_id` can make the report return no eligible customers. Cheapest discriminating test: add one eligible customer plus one blocked order whose `customer_id` is `NULL`, then compare both queries.
