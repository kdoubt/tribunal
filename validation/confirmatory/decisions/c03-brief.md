# Decision C03 - exclude a set of rows with NOT IN

A reporting query must return customers who have **no** order in a `blocked`
status. A dev writes:

```sql
SELECT * FROM customers c
WHERE c.id NOT IN (SELECT customer_id FROM orders WHERE status = 'blocked');
```

`orders.customer_id` is nullable (guest orders leave it NULL). **Question:** does
this query reliably return the intended customers under all data? Commit
("correct" / "not correct - fix X") and name the single biggest risk.
