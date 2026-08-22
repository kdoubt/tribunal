CLAIM: Choose A: shared database/shared schema with mandatory `tenant_id` and row-level security.
EVIDENCE: Hundreds of tenant databases multiply migrations, backups, monitoring, connection management, incident response, and version drift. Shared schema keeps operations and cross-tenant product analytics tractable.
CONFIDENCE: High.
FALSIFIER: A substantial share of target revenue requires contractual, regulatory, regional, encryption-key, or restore-level isolation that a shared database cannot satisfy.

CLAIM: A few very large tenants do not justify making every tenant database-isolated.
EVIDENCE: Resource quotas, workload management, partitioning, indexes beginning with `tenant_id`, and selective migration of exceptional tenants can address skew more economically than fleet-wide database-per-tenant operations.
CONFIDENCE: Medium-high.
FALSIFIER: One realistic large-tenant workload repeatedly causes unacceptable latency for unrelated tenants despite tested resource controls.

CLAIM: Row-level security must be the final enforcement boundary, not an optional application convention.
EVIDENCE: Application filters are easily omitted in new queries, background jobs, administrative paths, and reporting code; database-enforced policies materially reduce cross-tenant exposure.
CONFIDENCE: High.
FALSIFIER: The chosen database or operational tooling cannot enforce and continuously test tenant isolation across every access path.

VERDICT: A — biggest risk: a cross-tenant data leak caused by incorrect RLS, privileged bypass, or missing tenant context. Cheapest discriminating test: build an automated adversarial isolation suite using two tenants and every application/worker/admin role; attempt cross-tenant reads and writes while also running one large-tenant load profile. If complete isolation cannot be demonstrated reliably—or the load breaches another tenant’s SLO—A was the wrong choice.
