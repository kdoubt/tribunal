DECISION MEMO — C18: Feature Flags

CALL

A

Buy a managed feature-flag platform.

The organization needs three capabilities—not merely configuration toggles: reliable feature flags, controlled gradual rollouts, and experimentation. Building the control plane, low-latency SDKs, targeting rules, audit history, metrics integration, kill switches, caching, and failure-safe behavior would consume disproportionate engineering and operational attention for a 25-engineer team. These systems become production-critical infrastructure precisely when production is already failing.

Key tradeoffs and risks:

- Higher recurring cost and potential price growth as usage, environments, and seats expand.
- Vendor lock-in through proprietary SDK APIs, targeting rules, and experiment data.
- External dependency and data-governance concerns.
- Faster delivery, mature SDKs, auditability, permissions, rollout controls, and operational resilience.
- Less undifferentiated infrastructure work and fewer opportunities for subtle rollout or statistical errors.

Mitigate lock-in with a thin internal flag interface, centralized naming and lifecycle rules, conservative default values, and an exportable flag inventory. Require degraded operation from locally cached values so a vendor outage cannot take down request paths.

Single biggest risk: unmanaged adoption creates a permanent, expensive dependency—stale flags proliferate, vendor-specific logic leaks throughout the codebase, and migration becomes prohibitively difficult.

Cheapest wrong-choice test: run a 60-day pilot on one real service and one real gradual rollout. Track total monthly projected cost at full usage, developer time spent integrating and operating it, and whether the internal abstraction supports switching providers. The team chose wrong if the annualized platform cost exceeds the fully loaded cost of one engineer-month per year while the pilot uses only simple boolean/percentage flags and derives no material value from managed experimentation, audit, targeting, or governance features.
