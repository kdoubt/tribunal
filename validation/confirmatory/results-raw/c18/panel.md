## Decision memo (independent cross-vendor panel)

**Recommendation: A** - both independent seats, in isolation, chose A.

### Seat 1 position
VERDICT: A — Biggest risk: unpredictable vendor cost and lock-in as usage grows. Cheapest discriminating test: obtain a three-year quote using projected seats, contexts/events, environments, and growth, then compare it with a realistic internal estimate including at least 0.5 engineer-year to build and 0.25 engineer-year annually to operate. Choose B only if the managed cost is materially higher and a two-week prototype proves deterministic rollout, cached failure behavior, auditability, and metric attribution with acceptable maintenance.

### Seat 2 position
VERDICT: A — Biggest risk: locking into LaunchDarkly-class price/complexity for boolean+% use. Cheapest test: 90 days of usage. If they never run an experiment, flags stay env-level on/off, and the invoice exceeds ~one senior engineer-week, they overbought (or bought the wrong vendor).
