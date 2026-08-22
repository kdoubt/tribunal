DECISION MEMO — C16 Multitenancy data model

CALL
A

Shared DB, shared schema, `tenant_id` on every table, Postgres RLS as the enforcement layer. Hundreds of tenants is well inside this model. The few whales are a capacity problem, not a tenancy problem. Start isolated and you buy a fleet-ops tax you will pay on every migration, backup, pool, and admin query for the life of the product.

Tradeoffs
A: one schema, one migration, cheap pooling, trivial cross-tenant billing/support/analytics. Isolation is logical. Noisy neighbors share disks, CPU, and caches. A missed `tenant_id` predicate is a data-leak.
B: blast radius and restore are per customer; some enterprise contracts love it. You operate hundreds of databases: migrate N times, pool N ways, lose cheap cross-tenant queries, hit practical DB-count and connection-count ceilings. Extracting one whale from A later is painful. Collapsing B into A later is worse.

Biggest risk
Cross-tenant data exposure: a job, export, search, admin tool, or raw SQL path that bypasses RLS or forgets `tenant_id`. That is an existential trust failure, not an SLO miss.

Wrong-choice test
Stand up two tenants. Run a cheap isolation suite on every read/write surface (API, jobs, search, exports, webhooks, admin, migrations). Any cross-tenant row is an implementation fail, not a model fail.

Model-fail criterion: on a shared cluster, drive 1–2 synthetic whales to ~20–50× median volume. If small-tenant p99 cannot hold SLO without dedicated hardware, and extracting those whales costs more than a year of B’s ops, A was wrong. Secondary kill: if dedicated-DB isolation is a contractual must-have on a majority of the first ~20 enterprise deals, switch. Until one of those fires, do not build B.
