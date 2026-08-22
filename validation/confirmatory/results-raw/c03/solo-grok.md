**CALL:** Not correct — replace `NOT IN` with `NOT EXISTS` (or filter NULLs out of the subquery).

**Why:** `NOT IN` is three-valued. With a nullable `orders.customer_id`, the subquery can return `NULL`. Then for every customer `id` *X*:

`X NOT IN (…, NULL)` ≡ `X <> a AND X <> b AND … AND X <> NULL`

`X <> NULL` is `UNKNOWN`. `UNKNOWN AND …` is `UNKNOWN`. `WHERE` keeps only `TRUE`. One blocked guest order (`customer_id` NULL) makes the predicate unknown for **every** row, so the report returns **no customers** — including those who never had a blocked order.

That is not “sometimes missing a few rows.” It is a silent empty (or near-empty) result whenever any blocked row has a null `customer_id`. Guest orders make that data state expected, not exceptional.

`NOT EXISTS` is the intended semantics and is NULL-safe:

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

Equality to `c.id` never matches NULL, so guest blocked orders do not poison the set. `LEFT JOIN … WHERE o.customer_id IS NULL` is equivalent. `NOT IN (SELECT customer_id FROM orders WHERE status = 'blocked' AND customer_id IS NOT NULL)` also works, but only if every future writer remembers the filter.

**Key risks**
- One NULL in the subquery zeros the whole report.
- The failure is silent: valid SQL, no error, empty/wrong output.
- `SELECT *` plus this predicate will be copied into other reports.
- Optimizer differences vs `NOT EXISTS` are secondary; correctness is not.

**Single biggest risk:** A single blocked guest order (`customer_id` NULL) makes the query return **zero customers**.

**Cheapest disproof:** Insert one `orders` row: `customer_id = NULL`, `status = 'blocked'`. If the original query returns no (or far fewer) customers than `NOT EXISTS` on the same data, the team chose wrong. Criterion: results must match `NOT EXISTS` with and without that NULL row.
