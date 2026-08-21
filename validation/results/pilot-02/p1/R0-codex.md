CLAIM: B matches the actual complexity and scale.  
EVIDENCE: Thousands of orders/day is routine Postgres load. A transactional update can modify `orders` and append a full before/after snapshot to `order_history` atomically, supporting audit and “state at T” with a simple `(order_id, changed_at)` index.  
CONFIDENCE: 0.94  
FALSIFIER: Requirements demand reconstructing many derived states from business events—not merely retrieving historical order snapshots.

CLAIM: Event sourcing imposes substantial correctness machinery unrelated to the stated needs.  
EVIDENCE: A requires event schema evolution, deterministic replay, projection checkpoints/rebuilds, idempotent consumers, concurrency/version handling, and operational procedures for projection lag or corruption. The team has no experience with these mechanisms.  
CONFIDENCE: 0.92  
FALSIFIER: A production-grade event store, projection framework, and experienced event-sourcing lead already exist.

CLAIM: CRUD plus audit can provide a reliable trail if integrity is enforced in the database.  
EVIDENCE: A Postgres trigger can capture every mutation—including ad hoc SQL—in the same transaction and revoke application permissions to update/delete history. Full-row snapshots make temporal reads independent of historical application code.  
CONFIDENCE: 0.88  
FALSIFIER: The audit trail must be cryptographically tamper-evident against privileged database administrators or span multiple independently written stores.

CLAIM: Event sourcing does not automatically make historical truth easy.  
EVIDENCE: Replay under today’s code may differ from original behavior after event and projection logic evolve; preserving reproducible state requires versioned semantics and disciplined migrations. Snapshot history records the materialized state directly.  
CONFIDENCE: 0.87  
FALSIFIER: The business explicitly needs retroactive reinterpretation of old facts under new projection rules.

VERDICT: **B — CRUD + explicit audit log.**

Single biggest risk: the “audit” path silently misses mutations or records insufficient context, producing false confidence.

Cheapest discriminating test: build one representative order workflow and require a single SQL query to reproduce the order at arbitrary timestamps while tests mutate through both the application and direct SQL. If this cannot remain atomic, complete, and understandable—or temporal requirements require replaying business meaning rather than selecting snapshots—the team picked wrong and should choose A.
