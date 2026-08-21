CLAIM: No—binary64 cannot exactly represent most decimal currency fractions, including 0.01.  
EVIDENCE: IEEE-754 binary floating-point represents values as binary fractions; repeated postings and arithmetic introduce representation and rounding error.  
CONFIDENCE: 0.99  
FALSIFIER: Every supported amount and intermediate result is provably restricted to exactly representable binary fractions.

CLAIM: Summing or netting millions of float rows can produce order-dependent, non-associative results.  
EVIDENCE: In IEEE-754 arithmetic, `(a+b)+c` may differ from `a+(b+c)`; database query plans, parallel aggregation, and posting order can therefore alter the final low-order digits.  
CONFIDENCE: 0.99  
FALSIFIER: The database guarantees exact, order-independent aggregation with a defined decimal rounding policy despite float storage.

CLAIM: “15–16 significant digits” is not an adequate ledger invariant.  
EVIDENCE: Double precision is relative, while ledgers require exact minor-unit conservation and deterministic equality—debits must equal credits—not merely a small numerical error. Large balances also reduce the spacing between representable values relative to cents.  
CONFIDENCE: 0.99  
FALSIFIER: The product explicitly permits bounded approximate balances and never relies on exact equality, reconciliation, audit reproduction, or cent-level precision at its maximum magnitude.

CLAIM: Use scaled integers or fixed-point decimal with explicit currency precision and rounding rules.  
EVIDENCE: An `int64` count of the smallest supported unit gives exact, associative addition within its checked range; alternatively, `DECIMAL/NUMERIC(p,s)` provides exact base-10 representation. Currency must be stored separately because scales vary and some instruments require sub-minor units.  
CONFIDENCE: 0.98  
FALSIFIER: The required amount range and scale cannot fit the selected integer/decimal schema and no wider exact representation is available.

VERDICT: **NO** — the biggest risk is silent, order-dependent rounding drift that breaks exact balancing and reconciliation; fix it with checked scaled integers (preferred when scale is controlled) or fixed-point `DECIMAL/NUMERIC`, plus explicit per-currency scale and rounding policies.
