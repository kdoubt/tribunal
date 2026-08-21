# C06 SEALED
- correct_call: **NOT sufficient.** Synchronized exponential backoff without
  **jitter** makes all clients retry at the same instants (1s, 2s, 4s...),
  producing retry **thundering-herd** waves that hammer the recovering service.
  Backoff needs randomized jitter; ideally also a circuit breaker / retry budget.
- oracle: AWS Architecture Blog "Exponential Backoff And Jitter"; Google SRE
  (retry amplification).
- fix: add full/decorrelated jitter; cap concurrent retries (token bucket /
  circuit breaker); don't retry non-idempotent ops blindly.
- must_catch: [synchronized retries without jitter = herd; add jitter; circuit breaker / retry budget; retry amplification on a degraded dependency]
- landmine: "sufficient, backoff spreads load" / omitting jitter.
