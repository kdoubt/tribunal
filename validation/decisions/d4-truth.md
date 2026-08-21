# D4 sealed rubric - SEALED (arms never see this)

- **difficulty:** control (well-known; all arms expected to get it right)
- **correct_call:** **NO.** Binary IEEE-754 cannot represent most finite decimal
  fractions exactly (0.1, 0.2, 0.10 all recur in binary), so `0.1 + 0.2 !=
  0.3`. Errors accumulate when summing millions of rows and violate the exact
  equality/reconciliation an accounting ledger requires.
- **oracle:** IEEE-754 / "What Every Computer Scientist Should Know About
  Floating-Point"; PostgreSQL docs warn `double precision` is inexact and
  recommend `numeric`/`decimal` for money.
- **correct fix:** store integer **minor units** (cents/pennies as `bigint`), or
  an exact fixed-point `NUMERIC/DECIMAL` type; do money math in that exact type.
- **must_catch:**
  1. binary float can't represent decimal cents exactly;
  2. rounding error accumulates over many sums (reconciliation breaks);
  3. use integer minor units or `DECIMAL`/`NUMERIC` (exact) instead.
- **landmine (confident wrong answers):** "yes - a double has 15-16 digits, more
  than enough for cents"; "just round to 2 decimals on display" (does not fix
  accumulated internal drift).
