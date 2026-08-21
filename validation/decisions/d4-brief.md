# Decision D4 - money representation

A double-entry accounting ledger for a fintech app must store monetary amounts
(USD, EUR, etc.) and sum/net them across millions of rows. The proposal: store
each amount as a 64-bit IEEE-754 floating-point number (`double` / `float8`),
because doubles are fast, native everywhere, and give ~15-16 significant decimal
digits - far more than cents require.

**Question.** Is a 64-bit float the right storage type for these ledger amounts?
Commit to a call - "yes" or "no" - and name the single biggest risk (and the
fix, if any).
