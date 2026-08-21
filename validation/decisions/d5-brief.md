# Decision D5 - sequential invoice numbers

Invoices need a human-friendly, gapless-ish sequential number
(`INV-1`, `INV-2`, …) separate from the surrogate primary key. The proposed
implementation, run from the application on every invoice creation under
concurrent web requests:

```sql
SELECT COALESCE(MAX(invoice_no), 0) + 1 FROM invoices;   -- app reads next number
-- app then inserts the new invoice with that invoice_no
INSERT INTO invoices (..., invoice_no) VALUES (..., $next);
```

There is currently no unique constraint on `invoice_no`.

**Question.** Under concurrent invoice creation, does this reliably produce
**unique** invoice numbers? Commit to a call - "yes" or "no" - and name the
single biggest risk (and the fix, if any).
