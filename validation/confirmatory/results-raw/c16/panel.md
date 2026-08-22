## Decision memo (independent cross-vendor panel)

**Recommendation: A** - both independent seats, in isolation, chose A.

### Seat 1 position
VERDICT: A — biggest risk: a cross-tenant data leak caused by incorrect RLS, privileged bypass, or missing tenant context. Cheapest discriminating test: build an automated adversarial isolation suite using two tenants and every application/worker/admin role; attempt cross-tenant reads and writes while also running one large-tenant load profile. If complete isolation cannot be demonstrated reliably—or the load breaches another tenant’s SLO—A was the wrong choice.

### Seat 2 position
**VERDICT: A** — Biggest risk: a missed `tenant_id`/RLS predicate (cross-tenant leak) plus a whale melting the shared cluster. Cheapest test: replay the largest tenant’s volume/QPS against a shared cluster with dummy small tenants; if small-tenant p99 misses SLO, or any near-term contract requires a dedicated DB, A is wrong (use hybrid, not full B).
