**CALL: No — never use IEEE-754 floating point as the authoritative storage type for ledger amounts.**

**Reasoning:** Most decimal currency values are not exactly representable in binary floating point. Tiny errors accumulate during aggregation and netting, so mathematically equal debits and credits can compare unequal. Accounting requires exact, deterministic decimal arithmetic and invariant preservation—not merely enough significant digits.

**Single biggest risk:** The ledger silently fails to balance, producing incorrect balances, reconciliation discrepancies, or materially wrong financial reports.

**Fix:** Store amounts as fixed-point integers in each currency’s smallest supported unit (for example, cents or a documented finer internal scale) using `BIGINT`, with explicit currency and scale. If variable scales or unusually large ranges are required, use exact `DECIMAL/NUMERIC`; perform calculations with exact arithmetic throughout.
