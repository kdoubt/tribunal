## Decision memo (independent cross-vendor panel)

**Recommendation: A** - both independent seats, in isolation, chose A.

### Seat 1 position
VERDICT: A — Terraform. Biggest risk: HCL/module constraints become a productivity ceiling as app developers demand richer abstractions. Cheapest discriminating test: have both approaches implement and modify the same representative stack—including loops, conditional resources, reusable components, policy checks, and import of an existing resource—then compare reviewer comprehension and change lead time. Choose B instead if it is materially faster while its preview remains equally understandable to ops reviewers.

### Seat 2 position
**VERDICT: A** — Biggest risk: app developers bounce off HCL and spawn shadow Pulumi/CDK/scripts. Cheapest test: three non-ops engineers each add a realistic service using only published TF modules; if median wall-clock >1 day or 2/3 need ops to land it, A is wrong.
