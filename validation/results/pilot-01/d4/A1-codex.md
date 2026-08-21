**CALL: No — never use binary `float64` as the authoritative ledger amount.**

**Reasoning:** IEEE-754 binary64 is inherently inexact for most decimal fractions, including common monetary values such as `0.01` and `0.10`. Its 15–16 significant digits describe approximate precision, not exact decimal representation. Repeated summation across millions of rows can accumulate rounding error, and floating-point addition is order-dependent. PostgreSQL explicitly classifies `double precision` as inexact and recommends `numeric` when exact storage and calculations—specifically monetary amounts—are required. [PostgreSQL numeric types](https://www.postgresql.org/docs/15/datatype-numeric.html), [IEEE 754-2019](https://standards.ieee.org/ieee/754/6210/)

**Single biggest risk:** A balanced double-entry ledger can produce nonzero residuals or inconsistent totals depending on aggregation order, undermining reconciliation, equality checks, and auditability.

**Fix:** Store amounts as exact fixed-point values:

- Prefer a signed 64-bit integer in the currency’s explicitly defined smallest accounting unit, with overflow limits enforced.
- Use `NUMERIC/DECIMAL(p,s)` when currencies, instruments, fees, or exchange calculations require varying or greater decimal scale.
- Store the currency code alongside the amount and never sum different currencies without an explicit conversion entry.

Floating point is acceptable for derived analytics, but not as the ledger’s source of truth.
