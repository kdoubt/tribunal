DECISION MEMO — C11

A

Use PostgreSQL with a relational product core and JSONB for category-specific attributes. At ~1M products, scale is well within PostgreSQL’s range, the team already operates it confidently, and inventory/order joins remain straightforward. JSONB provides sufficient schema flexibility without adding another primary datastore’s operational and consistency model.

Key tradeoffs/risks:

- Faceting across heterogeneous JSONB attributes can produce complex queries, large or specialized indexes, and uneven performance.
- JSONB makes weak attribute governance tempting; inconsistent names, types, and units will degrade filtering quality.
- PostgreSQL search is adequate for basic needs, but sophisticated relevance, typo tolerance, and high-dimensional faceting may eventually require a dedicated search index.
- A document database improves document-shaped ergonomics, but does not eliminate the need for attribute normalization or a search engine, while making relational workflows and team operations harder.

Single biggest risk: dynamic faceting becomes too expensive or operationally awkward in PostgreSQL as the number and cardinality of filterable attributes grow.

Cheapest wrong-choice test: load a representative sample—ideally 100k–1M products with real category skew—and benchmark the 20 most important browse queries, including multi-select filters, high-cardinality facets, counts, sorting, and inventory availability. If properly indexed PostgreSQL cannot sustain the agreed p95 latency under expected peak concurrency without category-specific schema/index proliferation that the team considers unmaintainable, the choice is wrong. A practical criterion: fail A if representative facet queries cannot meet the product’s p95 target after one focused indexing iteration.
