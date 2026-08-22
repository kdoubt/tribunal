**CLAIM 1:** At hundreds of tenants, shared schema is the cheaper default.

**EVIDENCE:** One migration path, one connection pool, one backup/restore, one vacuum/upgrade. DB-per-tenant makes those O(N). Hundreds of idle small DBs still cost RAM, file handles, and connections; B2B SaaS at this scale almost always starts shared-schema.

**CONFIDENCE:** 0.85

**FALSIFIER:** End-to-end migrate+backup+monitor per tenant stays minutes, and a pooler keeps total connections bounded as N grows to the planned hundreds.

**CLAIM 2:** “A few very large” is an exception, not a reason to tax every small tenant with a dedicated DB.

**EVIDENCE:** Whales cause I/O and lock contention, but extracting 2–5 tenants (or partitioning/rate-limiting them) is a known move. Running hundreds of tiny databases for 5-seat customers wastes capacity. Merging later is harder than splitting a whale out.

**CONFIDENCE:** 0.80

**FALSIFIER:** Dedicated-DB is in the standard enterprise contract, or “few large” is actually a large fraction of tenant count, not load.

**CLAIM 3:** RLS + `tenant_id` is enough isolation for typical B2B SaaS, not for “own database” mandates.

**EVIDENCE:** Postgres RLS is production-grade if every query path is forced through it. Security questionnaires that demand a dedicated DB/VPC will not treat RLS as equivalent. App-layer misses (`tenant_id` omitted, superuser bypass, connection reuse) are the real leak path.

**CONFIDENCE:** 0.75

**FALSIFIER:** Top target customers require a dedicated DB, or a review of hot queries finds unscoped access that RLS would not have caught.

**CLAIM 4:** Cross-tenant work (billing, support, analytics, flags) is native in A and a warehouse project in B.

**EVIDENCE:** Shared schema lets the control plane query in place. Per-tenant DBs push that to N connections, FDW, or ETL. That pain is weekly, not theoretical.

**CONFIDENCE:** 0.80

**FALSIFIER:** A warehouse is already the source of truth and product DBs are never queried cross-tenant.

**VERDICT: A** — Biggest risk: a missed `tenant_id`/RLS predicate (cross-tenant leak) plus a whale melting the shared cluster. Cheapest test: replay the largest tenant’s volume/QPS against a shared cluster with dummy small tenants; if small-tenant p99 misses SLO, or any near-term contract requires a dedicated DB, A is wrong (use hybrid, not full B).
