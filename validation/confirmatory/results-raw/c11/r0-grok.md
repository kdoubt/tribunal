**CLAIM 1:** At ~1M SKUs, Postgres is not the bottleneck; introducing Mongo mainly buys a second ops/consistency surface.
**EVIDENCE:** 1M catalog rows is well inside Postgres. Inventory/orders already want a relational store; a document catalog forces dual-write or app-side joins for “occasional” joins the brief already needs.
**CONFIDENCE:** 0.88
**FALSIFIER:** Measured join+filter p95 on Postgres JSONB exceeds SLA while a Mongo+join-in-app path meets it on the same fixture and traffic mix.

**CLAIM 2:** Heterogeneous attributes are a JSONB + typed core problem, not a reason to abandon SQL.
**EVIDENCE:** SKU, price, status, category, inventory keys stay relational; category-specific attrs live in JSONB with GIN/expression indexes and a check constraint or JSON Schema. That is the usual catalog-of-record pattern. “Schemaless” still needs validation and indexes.
**CONFIDENCE:** 0.85
**FALSIFIER:** Attribute shapes change so often that even JSONB + generated columns cannot ship a category without a migration, while Mongo validation + indexes ship the same change in one deploy.

**CLAIM 3:** Faceting is the real product risk, and neither engine is a search engine.
**EVIDENCE:** Arbitrary multi-facet AND/OR + sort is where JSONB GIN and Mongo aggregation both get ugly. The discriminating layer is usually a search index (or precomputed facet tables), not the document vs row model.
**CONFIDENCE:** 0.80
**FALSIFIER:** Top-10 facet queries stay <100ms p95 on one store with only native indexes, and the other cannot without a sidecar.

**CLAIM 4:** Team fluency is a first-order cost, not a soft preference.
**EVIDENCE:** They already run Postgres. Mongo HA, backups, and query planning are a new failure domain for no scale or join win.
**CONFIDENCE:** 0.82
**FALSIFIER:** The team already operates Mongo in prod at similar SLO, and Postgres JSONB hiring/ops is the scarcer skill.

**VERDICT: A** — Biggest risk: category/facet pages blow p95 on sparse, multi-key JSONB filters. Cheapest test: load 1M realistic products, GIN on the attr document, run the 10 worst live facet+sort queries; if p95 misses SLA with indexes you’d actually ship, add a search sidecar or revisit B—not before.
