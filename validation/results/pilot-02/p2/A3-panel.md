## Decision memo — service mesh for ~15 services: contested; decide with a 2-week spike

**Recommendation:** This is a genuine judgment call, and it reduces to one
measurable question. Do **not** adopt Istio-style traffic management now, and do
**not** assume "libraries will just cover it." Run the spike below; it chooses
between the two live options:

- **Adopt a narrow mesh now** (Linkerd; identity/mTLS + wire metrics only,
  retries OFF) — if uniform workload identity/policy cannot be achieved
  in-process.
- **Defer the mesh** (SPIFFE/SPIRE + cert-manager for identity, CI/admission-
  enforced retry/timeout libraries, OpenTelemetry) — if it can.

**Not in dispute (holds either way):**

- **OpenTelemetry is mandatory regardless.** A mesh gives uniform *transport*
  telemetry but does not emit application spans or rationalize metric semantics;
  it does not by itself close the tracing gaps or metrics sprawl.
- **Mesh retries must start OFF.** Mesh and app retries "work independently and
  therefore might conflict" (Istio docs), and multi-hop retries amplify load on
  a degraded dependency (Google SRE). Retry/timeout ownership must be singular.
- **Deferring only works with enforcement:** CI/admission must prove every
  east-west client uses current identity + resilience libraries. Libraries
  without that are theater — this is the strongest point for adopting.
- **An API gateway governs north-south only;** it does not police ordinary
  east-west calls.
- **The threshold is coordination cost, not a service-count magic number,** and
  **a mesh with no named owner (upgrade cadence, rollback runbook, resource
  budget) is a worse failure domain than today's inconsistency** — that
  ownership is a hard gate.

**The one surviving disagreement:** can this cluster's languages get uniform
workload identity + policy **in-process** (SPIRE/cert-manager + libraries), or
does at least one hop still need a **sidecar**? If a sidecar is required for
uniform identity, adopt a narrow Linkerd (mTLS-only); if not, the mesh's
remaining benefit does not justify a new failure domain at this size.

**Single biggest risk:** the mesh becomes a complex traffic-programming platform
— an opaque new failure domain that adds more toil than it removes. Contain it
by scoping to identity + metrics, retries off, and gating on a named owner.

**Cheapest test that decides it:** a **two-week spike on 3 representative
services (two languages if you have them):** implement mTLS/identity + one
enforced retry/timeout budget + one complete trace across every hop, once via
**narrow Linkerd** and once via **SPIRE/cert-manager + CI-enforced libraries**.
Pick the path needing **fewer operator hours and application changes** while
passing enforced mTLS, one full trace, and a controlled dependency-timeout
failure. **Defer** if identity/policy can be made uniform without a sidecar;
**adopt narrow Linkerd** if any hop still needs one.
