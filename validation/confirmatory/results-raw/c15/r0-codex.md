CLAIM: A public API’s dominant constraint is operational safety across unknown clients, which favors REST.  
EVIDENCE: HTTP methods, status codes, cache semantics, rate limiting, observability, gateways, and CDNs work naturally at resource/endpoint granularity. GraphQL concentrates heterogeneous workloads behind one endpoint, making cost control and diagnosis harder.  
CONFIDENCE: High.  
FALSIFIER: Production-like traffic shows GraphQL queries can be reliably costed, bounded, cached, and attributed with no material operational premium.

CLAIM: REST produces a simpler compatibility contract for third-party integrations.  
EVIDENCE: OpenAPI exposes explicit operations and payloads, supports broad tooling and SDK generation, and lets providers evolve additively or version breaking changes. GraphQL’s schema is typed, but field deprecation and nullability changes still require disciplined compatibility management.  
CONFIDENCE: Medium-high.  
FALSIFIER: Target developers overwhelmingly need frequent cross-resource query composition that REST cannot serve without proliferating endpoints or round trips.

CLAIM: GraphQL’s flexibility transfers complexity—and potential abuse—to the server.  
EVIDENCE: Arbitrary nesting, aliases, fan-out, and expensive resolver combinations require depth limits, complexity budgets, persisted queries, batching, and careful authorization at field level. This burden is especially risky with untrusted public clients.  
CONFIDENCE: High.  
FALSIFIER: The domain is naturally graph-shaped, resolver costs are predictable, and strict query controls preserve most client flexibility.

CLAIM: REST does not preclude serving diverse clients efficiently.  
EVIDENCE: Sparse fieldsets, filtering, includes/expansions, pagination, bulk endpoints, and well-designed aggregates can cover common mobile, web, and integration needs while keeping supported query shapes explicit.  
CONFIDENCE: Medium.  
FALSIFIER: These mechanisms become an ad hoc query language or force sustained client-specific endpoint creation.

VERDICT: B — REST. Biggest risk: diverse clients may require so many cross-resource compositions that REST becomes chatty or accumulates bespoke endpoints. Cheapest discriminating test: model the ten most important client workflows and implement representative requests; choose GraphQL instead if REST consistently needs more than three sequential round trips or client-specific aggregate endpoints for a substantial fraction of them.
