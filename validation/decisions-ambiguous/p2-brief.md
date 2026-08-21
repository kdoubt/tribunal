# Decision P2 - adopt a service mesh now, or defer

A platform team runs **~15 microservices** on Kubernetes. They are feeling real
pain: inconsistent mTLS between services, no uniform ret/timeout/retry policy,
and patchy cross-service observability (tracing gaps, per-team metrics sprawl).
Someone proposes adopting a **service mesh** (Istio or Linkerd) now to solve all
three uniformly. Others argue a mesh is heavy operational surface for a 15-service
shop and the same problems can be solved with shared libraries + an API gateway +
an OpenTelemetry standard, deferring the mesh until ~40+ services.

- **A) Adopt a service mesh now** (Istio/Linkerd) as the uniform layer for mTLS,
  traffic policy, and telemetry.
- **B) Defer the mesh** - solve with libraries/sidecarless OTel + a gateway +
  policy conventions; revisit a mesh at larger scale.

No consensus; both are defensible at this size. **Commit to A or B**, name the
single biggest risk of your choice, and give the cheapest test or leading
indicator that would tell this team they chose wrong.
