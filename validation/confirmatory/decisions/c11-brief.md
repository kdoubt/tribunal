# Decision C11 - SQL vs document DB for a new product catalog

A team is building a **product catalog + search** for a mid-size e-commerce site: heterogeneous product attributes (varies wildly by category), moderate scale (~1M products), needs filtering/faceting and occasional relational joins to inventory/orders. The team knows Postgres well. Choose the primary datastore:

- **A) PostgreSQL** (JSONB for variable attributes + relational core).
- **B) A document database** (e.g. MongoDB) for the flexible-schema catalog.

Commit to A or B, name the single biggest risk, and give the cheapest test/criterion that would tell them they chose wrong.
