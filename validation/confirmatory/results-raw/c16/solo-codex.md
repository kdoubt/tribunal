DECISION MEMO — C16 Multitenancy Data Model

CALL

A

Use a shared database and shared schema, with mandatory `tenant_id`, database-enforced row-level security (RLS), tenant-aware indexes, and tenant-scoped uniqueness constraints.

Key tradeoffs and risks:

- A minimizes operational cost and complexity across hundreds of tenants: one migration path, one connection strategy, efficient resource pooling, simpler analytics, and fewer backup/restore workflows.
- RLS provides strong isolation only when the database—not application convention—derives tenant context and enforces access. Administrative roles, background jobs, migrations, and support tooling require explicit controls because privileged paths can bypass RLS.
- A few very large tenants may create noisy-neighbor effects, oversized indexes, long-running queries, or disproportionate storage and maintenance load.
- Per-tenant restore, deletion, residency, encryption keys, and data export are harder in a shared physical database.
- B offers a cleaner isolation and failure boundary, but hundreds of databases impose substantial fleet-management overhead: schema drift, migrations, monitoring, connection pressure, capacity fragmentation, and cross-tenant reporting.

Single biggest risk:

A cross-tenant data exposure caused by an incorrectly scoped query, policy, privileged process, or operational tool. This is an existential trust and compliance failure, outweighing ordinary performance concerns.

Cheapest falsification test:

Build an automated isolation suite before committing the schema. Create two tenants with overlapping identifiers, then exercise every data-access path—including joins, bulk jobs, exports, support tooling, and direct SQL—while authenticated as one tenant. Require that unauthorized cross-tenant reads and writes fail at the database layer, even when application filters are intentionally omitted.

They chose wrong if any normal application or operational role can access another tenant’s rows by omitting or altering `tenant_id`, or if a committed large customer has a non-negotiable requirement for dedicated physical databases that cannot be met by selectively promoting exceptional tenants later.
