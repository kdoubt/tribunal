**CLAIM 1:** SAML/SSO is a long-tail compatibility product, not a library feature.  
**EVIDENCE:** SP vs IdP-initiated, ACS/metadata, cert rotation, JIT, group mapping, and Okta/Azure/Google quirks routinely consume months; WorkOS exists because that matrix is the job. A 20-engineer B2B team does not have a dedicated identity squad.  
**CONFIDENCE:** 0.85  
**FALSIFIER:** One engineer ships production SSO against Okta + Entra + Google Workspace (IdP- and SP-initiated, cert rollover, JIT) in ≤5 days with no vendor and no customer-blocking bugs.

**CLAIM 2:** Auth is not the product; opportunity cost dominates.  
**EVIDENCE:** Email/password, MFA (TOTP/WebAuthn/recovery), session/CSRF, lockout, and recovery are high-severity, ongoing CVE work. At ~20 engineers, that work displaces roadmap that closes deals.  
**CONFIDENCE:** 0.8  
**FALSIFIER:** Identity work stays under ~0.25 FTE for 12 months including MFA policy per org and incident response, with no delayed enterprise close.

**CLAIM 3:** For this shape, managed TCO is usually below one engineer-year.  
**EVIDENCE:** B2B MAU is org-gated, not consumer-scale. Auth0 can get expensive, but WorkOS/Clerk-class SSO pricing for hundreds of companies is typically well under a fully loaded engineer; buy also buys IdP certification and SLA.  
**CONFIDENCE:** 0.7  
**FALSIFIER:** 24-month vendor quote (seats/MAU + SSO add-ons) exceeds one fully loaded FTE *and* self-host ops stay under 0.25 FTE.

**CLAIM 4:** Self-hosting does not remove identity risk; it relocates it to upgrades, HA, backups, and your incident page.  
**EVIDENCE:** Keycloak/Ory/Authentik still need patch cadence, secret storage, and on-call. Customers already hold identity in their IdP once SSO is on.  
**CONFIDENCE:** 0.75  
**FALSIFIER:** Data-residency or air-gap is a hard procurement block that no managed IdP can satisfy.

**VERDICT: A** — Biggest risk: Auth0-style MAU/SSO pricing or missing tenant workflows (SCIM, org MFA, residency) after lock-in. Cheapest test: price the vendor at 24-month projected orgs/MAU; if it exceeds ~0.5 fully loaded FTE *and* a 3-day OSS spike completes real Okta+Entra login+JIT+MFA, you bought the wrong thing.
