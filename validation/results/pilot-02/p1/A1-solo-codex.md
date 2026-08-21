## Decision memo — Choose B: CRUD + explicit audit log

**Recommendation:** Use a conventional Postgres `orders` model with an append-only `order_history` table. Write the order update and its history record in the same database transaction. Store actor, timestamp, reason/correlation ID, version, and sufficient before/after state—preferably a full order snapshot per version at this scale.

**Why B:** It satisfies the stated requirements without making event replay, projections, event-schema evolution, idempotency, and eventual consistency part of the service’s core operating model. At thousands of orders per day, snapshot storage is inexpensive, temporal lookup is straightforward, and the team can use its existing Postgres/REST strengths. Event sourcing’s flexibility is not free: it moves complexity from writes into nearly every read, migration, repair, and debugging workflow.

**Decision-relevant tradeoffs and risks:**

- B provides reliable auditability only if mutations cannot bypass history capture. Enforce transactional writes through one path, database permissions, append-only controls, and invariant tests.
- Point-in-time reconstruction is less elegant than replaying events, but versioned snapshots make it simple: select the latest history row at or before time T.
- B records state transitions well but may not preserve rich business intent unless audit entries include operation type and reason—not merely a JSON diff.
- A would be stronger if event streams must drive many independent projections, exact historical replay is central to the product, or business behavior depends heavily on event sequence. Those needs are not established here.

**Single biggest risk:** The mutable order and audit history diverge because a code path, migration, or administrative operation updates `orders` without atomically recording history. That would undermine the primary requirement while appearing healthy operationally.

**Cheapest falsification test:** Implement one representative order lifecycle—creation, payment, partial fulfillment, cancellation, refund, and an administrative correction—and answer the expected “state at T” questions from history. Then list credible near-term consumers that require replayable domain events rather than snapshots. If reconstruction is ambiguous, or at least two committed consumers require independent replay/projections from the authoritative order stream, B is the wrong choice; adopt event sourcing before broad rollout.
