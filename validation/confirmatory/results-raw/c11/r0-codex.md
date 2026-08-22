CLAIM: PostgreSQL is the stronger primary datastore.
EVIDENCE: JSONB handles category-specific attributes while relational columns preserve stable fields, constraints, transactions, and straightforward joins to inventory/orders. The team’s existing Postgres expertise lowers delivery and operational risk.
CONFIDENCE: High.
FALSIFIER: Representative catalog queries cannot meet latency targets despite suitable JSONB indexes, generated columns, and schema tuning.

CLAIM: One million products is moderate for PostgreSQL.
EVIDENCE: PostgreSQL can comfortably store and index this volume; workload shape, index cardinality, and facet complexity matter more than raw document count.
CONFIDENCE: High.
FALSIFIER: Production-like concurrency and write rates cause unacceptable contention or resource costs on realistically sized hardware.

CLAIM: A document database does not eliminate the hard part of heterogeneous filtering.
EVIDENCE: Dynamic attributes still require deliberate indexing, normalization rules, and controls against index proliferation. MongoDB’s flexible schema shifts rather than removes data-model governance.
CONFIDENCE: High.
FALSIFIER: The real workload is overwhelmingly whole-document access with rapidly changing shapes, minimal joins, and indexing patterns demonstrably simpler and cheaper in the document database.

CLAIM: Search/faceting may eventually require a dedicated search engine, independent of the primary datastore.
EVIDENCE: Complex relevance, typo tolerance, aggregations, and high-cardinality facets commonly exceed what either primary database should optimally serve.
CONFIDENCE: Medium-high.
FALSIFIER: PostgreSQL full-text and indexed filters satisfy all measured search requirements with ample headroom.

VERDICT: A — Biggest risk: uncontrolled JSONB attribute diversity creates excessive indexes and slow, unpredictable facet queries. Cheapest discriminating test: load a representative category mix and query distribution, then benchmark the ten hardest filters/facets plus required joins at target concurrency; choose B only if PostgreSQL misses the agreed p95 latency after basic, production-plausible indexing and MongoDB clearly passes the same test.
