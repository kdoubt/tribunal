**CLAIM:** A public API’s consumers are untrusted, heterogeneous, and tool-driven; REST + OpenAPI is the default contract those clients, gateways, SDKs, and low-code tools already speak.  
**EVIDENCE:** Stripe, Twilio, Slack, and most partner ecosystems ship REST as the primary surface; GraphQL, when present (GitHub, Shopify), is a second surface for power users, not the on-ramp.  
**CONFIDENCE:** 80  
**FALSIFIER:** A representative sample of target integrators (languages, Zapier/Make, API gateways) cannot complete a core flow from an OpenAPI spec without custom GraphQL clients.

**CLAIM:** GraphQL’s query language on a public surface creates cost, cache, and abuse problems REST already solved with HTTP.  
**EVIDENCE:** One POST endpoint defeats CDN/ETag caching; nested queries enable complexity DoS; “one request” is not one resource, so fair rate limits need a query-cost model most teams underbuild.  
**CONFIDENCE:** 85  
**FALSIFIER:** You can publish a complexity/depth budget, HTTP-cacheable queries, and per-field rate limits that cover real third-party queries without a dedicated GraphQL ops team.

**CLAIM:** Diverse, evolving needs do not require GraphQL as the *primary* style; they require sparse fieldsets, includes, filters, webhooks, and additive versioning.  
**EVIDENCE:** JSON:API / OData-style `fields`/`include`, plus webhooks, cover most over/under-fetch cases without handing strangers a query planner. Schema evolution still breaks clients; “no versioning” is marketing.  
**CONFIDENCE:** 75  
**FALSIFIER:** Typical partner jobs need arbitrary nested graphs that cannot be expressed as 1–2 resource calls plus webhooks without exploding the REST surface.

**CLAIM:** First-party web/mobile overfetch is a product of *your* clients; a public API should optimize for stable, documentable resources, not app-by-app query shapes.  
**EVIDENCE:** Internal BFF/GraphQL can sit behind REST; reversing that (GraphQL public, REST later) strands every integrator on a query language they didn’t ask for.  
**CONFIDENCE:** 70  
**FALSIFIER:** The majority of early volume is first-party apps you control, with third parties a thin tail.

VERDICT: B — Biggest risk: chatty/overfetched integrations and endpoint sprawl as nested partner use cases grow. Cheapest test: implement the 5 highest-value third-party jobs against a draft OpenAPI; if median job needs ≥4 round trips or discards >50% of bytes, REST as the *only* public surface is wrong (add GraphQL later, don’t lead with it).
