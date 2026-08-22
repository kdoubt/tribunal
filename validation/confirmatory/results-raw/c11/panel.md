## Decision memo (independent cross-vendor panel)

**Recommendation: A** - both independent seats, in isolation, chose A.

### Seat 1 position
VERDICT: A — Biggest risk: uncontrolled JSONB attribute diversity creates excessive indexes and slow, unpredictable facet queries. Cheapest discriminating test: load a representative category mix and query distribution, then benchmark the ten hardest filters/facets plus required joins at target concurrency; choose B only if PostgreSQL misses the agreed p95 latency after basic, production-plausible indexing and MongoDB clearly passes the same test.

### Seat 2 position
**VERDICT: A** — Biggest risk: category/facet pages blow p95 on sparse, multi-key JSONB filters. Cheapest test: load 1M realistic products, GIN on the attr document, run the 10 worst live facet+sort queries; if p95 misses SLA with indexes you’d actually ship, add a search sidecar or revisit B—not before.
