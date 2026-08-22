## Decision memo (independent cross-vendor panel)

**Recommendation: A** - both independent seats, in isolation, chose A.

### Seat 1 position
VERDICT: A — Biggest risk: vendor pricing/lock-in becoming punitive as users and enterprise connections grow. Cheapest discriminating test: obtain binding price estimates for the next three years using projected MAU, SSO connections, MFA, and support needs; choose B only if the vendor case materially breaks unit economics and a named internal owner can demonstrate production-ready SAML, MFA recovery, patching, audit, and on-call capability.

### Seat 2 position
**VERDICT: A** — Biggest risk: Auth0-style MAU/SSO pricing or missing tenant workflows (SCIM, org MFA, residency) after lock-in. Cheapest test: price the vendor at 24-month projected orgs/MAU; if it exceeds ~0.5 fully loaded FTE *and* a 3-day OSS spike completes real Okta+Entra login+JIT+MFA, you bought the wrong thing.
