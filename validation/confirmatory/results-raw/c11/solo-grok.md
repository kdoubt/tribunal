**DECISION MEMO — C11: Primary datastore for product catalog + search**

**CALL**
A

**Decision.** Use PostgreSQL as the system of record: relational tables for identity, pricing, inventory/order FKs, and category taxonomies; JSONB (+ GIN / expression indexes) for category-specific attributes. Search/faceting may later sit in a dedicated index, but that is not a reason to move the catalog off Postgres.

**Why A.** ~1M SKUs is not a scale problem. Heterogeneous attributes are a schema problem Postgres already solves with JSONB. The team already operates Postgres. Occasional joins to inventory and orders are cheap and consistent in one engine; they are the expensive, failure-prone part of a document store (app-side joins, `$lookup`, dual-write, weaker cross-entity integrity). Mongo’s flexibility is real; it does not pay for a second datastore, a second ops surface, and a worse join story at this size.

**Tradeoffs.** JSONB filters are less ergonomic than a native document model. Multi-attribute facet queries can need careful GIN/expression indexes and may still lose to a search engine on high-cardinality, many-AND facets. Attribute shape is not “schemaless” in practice—you still need conventions, validation, and index hygiene. You are not buying best-in-class search by picking A *or* B; if search is the product, neither DB is the long-term search plane.

**Single biggest risk.** Filter/facet p95 on multi-key JSONB predicates blows the interactive SLO (table scans or unselective GIN AND-combinations) while writes remain fine—so the catalog looks healthy until shoppers hit category pages.

**Cheapest wrong-choice test.** Before commit: load ~1M synthetic SKUs with production-like attribute cardinality and sparsity. Run the 10 hottest category filter+facet queries under a hard p95 (e.g. 150ms) with `EXPLAIN ANALYZE`. Fail the choice if any of those 10 cannot be made index-only / bitmap-AND without a sequential scan, or if adding the next two realistic filters forces a rewrite of the access path. That failure means you needed a search index (or, less likely, a document engine)—not that Mongo should have been the source of truth.
