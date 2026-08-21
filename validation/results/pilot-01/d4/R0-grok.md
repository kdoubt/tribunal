I’ll treat this as an independent panel seat: check whether the float storage claim holds under IEEE-754 and ledger invariants, then commit to a yes/no call.**CLAIM 1.** Binary64 cannot store most ISO 4217 amounts exactly, including ordinary USD cents.

**EVIDENCE.** IEEE-754 binary64 is a dyadic rational: `(-1)^s × 2^(e-1023) × (1 + m/2^52)`. `0.1` has a repeating binary expansion (`0.00011001100…₂`); the stored value is `0x1.999999999999ap-4` = `0.1000000000000000055511151231257827021181583404541015625`, not one tenth (Goldberg, *What Every Computer Scientist Should Know About Floating-Point Arithmetic*, 1991; Python docs §15). USD/EUR `0.01` fails the same way. ISO 4217 minor-unit exponents are decimal (USD=2, JPY=0, BHD/KWD=3), not powers of two.

**CONFIDENCE.** 1.0

**FALSIFIER.** A proof that every ledger amount is a finite dyadic (`k/2^n` with `n ≤ 52`), or a dump of `float8` rows whose decoded values equal the posted decimal strings bit-for-bit.

---

**CLAIM 2.** Sum/net over millions of rows is not a well-defined exact number, so the double-entry identity cannot be checked with equality.

**EVIDENCE.** IEEE-754 addition is commutative but not associative; `SUM(amount)` over `float8` depends on insertion order, parallel workers, and fused vs. rounded add (Goldberg §§1.3–1.4). A trial balance that must be exactly 0, and a debit/credit pair that must cancel, become “within a few ulps.” Parallel `SUM` and map-reduce nets will disagree by machine epsilon × magnitude. Banks and card networks post ISO 4217 minor units (Adyen/Stripe integer-cents convention); reconciling a drifted `float8` net against those files fails or requires a fudge epsilon, which is not an accounting control.

**CONFIDENCE.** 0.95

**FALSIFIER.** A production-scale mixed-cent ledger in `float8` whose `SUM(debits) − SUM(credits)` is exactly `0.0` under reordering, sharded reduction, and replay, with no rounding/epsilon step.

---

**CLAIM 3.** The “~15–16 significant digits, more than cents need” argument is the wrong invariant: money requires *absolute* minor-unit exactness, not relative precision.

**EVIDENCE.** 53-bit significand gives ~15.95 decimal digits of *relative* precision. Absolute error on a `$1e9` balance is on the order of `1e9 / 2^53 ≈ 1e-7` dollars (~0.01 cent) *per operation*, and grows with the reduction tree — enough to move a cent after enough posts, FX, splits, or interest. Integers in binary64 are exact only up to `2^53` (~9e15); that bound is for *integer cents stored as integers*, which is not what the proposal describes. The type that matches the domain is `int64` minor units (range `±9.2e18`) or `NUMERIC`/`DECIMAL` with currency-defined scale.

**CONFIDENCE.** 0.93

**FALSIFIER.** A written ledger requirement that specifies scientific (relative) precision rather than exact minor-unit amounts, *and* a proof that no FX, fee, split, or interest ever produces a non-dyadic.

---

**CLAIM 4.** Even the “store integer cents in a double” rescue does not make `float8` the right storage type.

**EVIDENCE.** Integer-valued binary64 is exact only for `|n| ≤ 2^53`. One FX rate, percentage fee, or 3-way split injects a fraction and the invariant is gone. Sub-cent products, BHD (3 decimals), and crypto-scale units blow the integer-cent story. `int64` minor units keep exact addition; `NUMERIC` keeps exact decimal scale. `float8` buys neither, and buys NaN/Inf, `-0.0`, and JSON/JS number hazards on the wire.

**CONFIDENCE.** 0.88

**FALSIFIER.** A closed ledger that (a) stores only integer cents, (b) never divides or applies a non-integer rate, and (c) never exceeds `2^53` in any running total — in which case `int64` is still the cheaper, exact type.

---

**VERDICT.** **NO** — do not store ledger amounts as `double`/`float8`. Biggest risk: silent, order-dependent rounding drift that breaks the debit=credit identity and bank reconciliation; the books no longer prove. Fix: `int64` ISO 4217 minor units (or `NUMERIC` with per-currency scale) and exact integer/decimal arithmetic for nets.
