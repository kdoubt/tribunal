**CLAIM 1:** The query is **not correct**. A single `NULL` in the subquery empties the result.

**EVIDENCE:** `x NOT IN (a, b, NULL)` is `x <> a AND x <> b AND x <> NULL`. `x <> NULL` is UNKNOWN, so the AND is UNKNOWN. `WHERE` keeps only TRUE rows, so **every** customer is filtered out once any blocked guest order exists.

**CONFIDENCE:** 0.99

**FALSIFIER:** On an engine that treats `NOT IN` as `NOT (x = ANY(...))` with NULLs dropped, or if `customer_id` is actually `NOT NULL`.

---

**CLAIM 2:** Intended semantics are “no *matching* blocked order for this customer.” Guest NULLs must not poison the predicate.

**EVIDENCE:** `NULL` is not a customer id. A blocked guest order should exclude nobody. `NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id AND o.status = 'blocked')` does that: `NULL = c.id` is UNKNOWN, so guests never match. Filtering `customer_id IS NOT NULL` inside `NOT IN` also works, but is easier to forget.

**CONFIDENCE:** 0.97

**FALSIFIER:** Product rule that “any blocked guest order hides all customers.”

---

**CLAIM 3:** This fails **silently and data-dependently**. Tests with only non-null `customer_id` pass; production guest traffic makes the report go empty.

**EVIDENCE:** No error, no warning. Empty report looks like “no eligible customers.” Classic `NOT IN` + nullable column trap.

**CONFIDENCE:** 0.95

**FALSIFIER:** CI that seeds a blocked order with `customer_id IS NULL` and asserts a non-empty customer list.

---

**CLAIM 4:** `NOT EXISTS` (or `LEFT JOIN ... WHERE o.id IS NULL`) is the fix; do not leave `NOT IN` on a nullable column.

**EVIDENCE:** Anti-join over equality never lets a NULL in the inner set turn the outer predicate UNKNOWN for unrelated rows. Index-friendly and matches the English rule.

**CONFIDENCE:** 0.93

**FALSIFIER:** Schema change making `orders.customer_id` `NOT NULL`, plus a check that no blocked row can have a null id.

---

**VERDICT:** **not correct — replace with `NOT EXISTS` (or `NOT IN` + `customer_id IS NOT NULL`).** Biggest risk: one blocked guest order silently returns **zero** customers. Cheapest test: insert one `orders` row `(customer_id=NULL, status='blocked')` with a customer who has no blocked orders; the customer must still appear.
