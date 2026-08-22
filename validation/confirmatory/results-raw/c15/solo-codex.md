DECISION MEMO — C15

CALL

B

Use REST with resource-oriented endpoints, OpenAPI, additive evolution, cursor pagination, sparse field selection, and explicit versioning only for breaking changes.

Key tradeoffs and risks

REST is the stronger default for a public API because its operational and adoption characteristics are predictable across languages, tooling, gateways, CDNs, security products, and third-party integrations. OpenAPI supports documentation, SDK generation, contract testing, and client onboarding without requiring consumers to adopt GraphQL-specific tooling or query semantics.

The cost is reduced client flexibility. Diverse clients may need multiple requests, receive unnecessary fields, or wait for new projections and aggregate endpoints. Endpoint proliferation and inconsistent resource modeling are real risks. Mitigate these with composable filters, includes/expansions, sparse fieldsets, batch endpoints, and disciplined API governance—not premature migration to a programmable query surface.

GraphQL would offer superior client-driven composition and schema discoverability, especially for deeply connected data. For a public API, however, it shifts substantial complexity into production controls: query-cost analysis, depth limits, resolver performance, authorization at field level, caching, observability, persisted queries, and abuse prevention. It also gives external clients freedom that becomes difficult to retract.

Single biggest risk

REST fails to serve materially different client data shapes efficiently, forcing excessive request fan-out and a growing collection of client-specific endpoints.

Cheapest falsification test

Before launch, model the ten highest-value workflows across web, mobile, and two representative third-party integrations. Implement them against the proposed REST contract and measure calls per workflow.

The team chose wrong if at least three workflows require either more than five sequential API requests to render one user-visible operation or a client-specific aggregate endpoint that cannot be expressed as a reusable domain resource. If that criterion is met despite includes, sparse fields, and batching, choose GraphQL.
