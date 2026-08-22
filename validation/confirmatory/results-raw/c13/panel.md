## Decision memo (independent cross-vendor panel)

**Recommendation: B** - both independent seats, in isolation, chose B.

### Seat 1 position
VERDICT: B — Biggest risk: discovering too late that durable replay and independent historical reprocessing are core requirements. Cheapest discriminating test: ask each consumer owner, “Must you be able to rebuild or reprocess from the complete order-event history after deployment?” If any critical consumer answers yes and retention/replay cannot be cheaply added, the team chose wrong.

### Seat 2 position
**VERDICT: B** Biggest risk: a later consumer needs durable historical replay (events, not current DB rows) and the queue has already dropped messages. Cheapest test: specify one dummy consumer that must rebuild last 30 days of *events* with the DB unavailable—if that is a real requirement, B is wrong; if backfill-from-orders is acceptable, B is right.
