## Decision memo (independent cross-vendor panel)

**Recommendation: B** - both independent seats, in isolation, chose B.

### Seat 1 position
VERDICT: B — biggest risk: ecosystem and debugging friction causes developers to evade the standard. Cheapest discriminating test: migrate one representative Go service with Python and Node consumers, require one backward-compatible schema evolution, then measure manual client code, bypasses, integration failures, and debugging time; choose A if gRPC produces more exceptions or materially slower diagnosis than generated OpenAPI/REST.

### Seat 2 position
**VERDICT: B** — Biggest risk: Python/Node/ops friction creates shadow JSON APIs and two standards. Cheapest test: one Go↔Python and one Node↔Go RPC with protobuf, buf breaking checks, and a live failed-call debug; if either team needs a JSON escape hatch or debug time is >2× REST, switch to A with mandatory OpenAPI codegen + breaking-change CI.
