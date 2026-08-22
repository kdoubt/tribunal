## Decision Memo — C03

**CALL: Not correct — replace `NOT IN` with correlated `NOT EXISTS`.**

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

**Single biggest risk:** One blocked guest order with `customer_id = NULL` causes the `NOT IN` predicate to evaluate as `UNKNOWN` for every otherwise eligible customer. The report can silently return zero rows—or omit all qualifying customers.

**Key risks:**

- SQL uses three-valued logic: `x NOT IN (..., NULL)` cannot establish that `x` differs from every value.
- The failure is data-dependent, so the query may appear correct until the first matching NULL enters the subquery.
- Adding `customer_id IS NOT NULL` inside the subquery would repair this instance, but `NOT EXISTS` directly expresses the requirement and remains safe if nullability assumptions change.

**Cheapest test / wrong-choice criterion:** Insert one blocked order whose `customer_id` is NULL alongside a customer with no blocked orders. If that customer disappears, the query is wrong.

Quick execution confirmed it: with customers `1, 2, 3`, a blocked order for customer `1`, and a blocked order with NULL customer, `NOT IN` returned no customers; `NOT EXISTS` correctly returned `2` and `3`.
