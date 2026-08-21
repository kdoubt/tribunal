## Decision Memo: Defer the Service Mesh

**Recommendation: B — defer the mesh.**

At ~15 services, the platform should standardize the problems directly: cluster-native mTLS, shared timeout/retry conventions, gateway policy, and sidecarless OpenTelemetry. Revisit a mesh when demonstrated coordination or enforcement costs exceed its operational cost—not at an arbitrary service count.

**Decision-relevant tradeoffs**

- **Lower operational burden:** A mesh adds control-plane upgrades, proxy behavior, resource overhead, certificate and policy debugging, and another failure domain. That cost is disproportionate for 15 services.
- **Faster near-term remediation:** Standard OTel instrumentation, dashboards, and explicit client policies address today’s gaps without a platform migration.
- **Less uniform enforcement:** Libraries and conventions can drift across languages and teams. They provide weaker guarantees than centrally enforced mesh policy.
- **Migration debt remains:** If the system grows substantially, adopting a mesh later will require another rollout and may expose assumptions embedded in libraries.
- **Avoid false safety:** Mesh retries can amplify outages and mTLS can complicate debugging. Centralization makes controls consistent, but does not make them automatically correct.

**Single biggest risk**

The team fails to enforce its “lighter” standards. Per-service exceptions accumulate, mTLS coverage remains incomplete, telemetry stays fragmented, and engineers repeatedly diagnose policy differences that a mesh would have eliminated.

**Cheapest test that would show this decision is wrong**

Within 90 days, automatically audit every service and require:

1. 100% authenticated service-to-service traffic,
2. ≥95% trace continuity across service boundaries, and
3. one centrally defined timeout/retry policy profile per protocol, with documented exceptions.

If more than two services fail any criterion for two consecutive monthly audits—or platform engineers spend more than one engineer-week per month chasing cross-service policy drift—the team chose wrong and should start a mesh pilot.
