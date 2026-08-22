DECISION MEMO — C15: Public API style

CALL
B

REST as the primary public contract (resource URLs, HTTP semantics, explicit versioning, OpenAPI). GraphQL may be a later, optional surface — not the platform default.

Tradeoffs
GraphQL wins on client-shaped payloads and fewer round-trips for web/mobile that share one evolving graph. That is real. For a *public* API it is the wrong default.

External integrators need a stable, curl-able, cacheable, rate-limitable contract. REST gives status codes, ETags/CDNs, idempotency keys, per-route quotas, and OpenAPI-generated SDKs that partners already know. GraphQL collapses that into one POST, so you re-implement caching (persisted queries), cost limits (query complexity), authz (field-level), and support (everyone writes a different query). Schema evolution without versions also leaks breaking changes to strangers you cannot force-upgrade.

Diverse needs are better met with sparse fieldsets, includes, webhooks, and versioned resources than with unbounded client queries. File uploads, webhooks, and batch jobs still look like REST even if you ship GraphQL.

Single biggest risk
Endpoint and version explosion: you paper over each integrator’s shape with new includes, custom routes, and v2/v3, then cannot freeze a contract.

Cheapest wrong-choice test
Before locking the style, take the five highest-value third-party jobs-to-be-done and implement them against a draft OpenAPI. Count sequential GETs and unused bytes. Ship REST only if each job is ≤2 round-trips with sparse fieldsets/`include`, unused payload <40%, and you can name a freezeable v1. After launch, if the top 20 partners each force a new nested include or custom endpoint per quarter, or median unused bytes stay >60%, the call was wrong — add GraphQL as a second surface, do not pretend REST is enough.
