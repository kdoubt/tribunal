## Decision memo (independent cross-vendor panel)

**Recommendation: B** - both independent seats, in isolation, chose B.

### Seat 1 position
VERDICT: B — REST. Biggest risk: diverse clients may require so many cross-resource compositions that REST becomes chatty or accumulates bespoke endpoints. Cheapest discriminating test: model the ten most important client workflows and implement representative requests; choose GraphQL instead if REST consistently needs more than three sequential round trips or client-specific aggregate endpoints for a substantial fraction of them.

### Seat 2 position
VERDICT: B — Biggest risk: chatty/overfetched integrations and endpoint sprawl as nested partner use cases grow. Cheapest test: implement the 5 highest-value third-party jobs against a draft OpenAPI; if median job needs ≥4 round trips or discards >50% of bytes, REST as the *only* public surface is wrong (add GraphQL later, don’t lead with it).
