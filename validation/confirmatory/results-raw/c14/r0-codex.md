CLAIM: Managed identity is the better default for a 20-engineer B2B SaaS.
EVIDENCE: Password security, MFA recovery, session management, SAML quirks, account linking, auditability, and incident response form an ongoing security product—not a one-time integration. Building diverts scarce engineers from differentiated work.
CONFIDENCE: High.
FALSIFIER: Authentication is itself core product IP, or regulatory/data-residency requirements cannot be met by credible vendors.

CLAIM: Enterprise SSO strongly favors buying.
EVIDENCE: Each customer’s SAML/IdP configuration creates support edge cases. Managed providers offer tested connectors, metadata handling, certificate rotation, diagnostics, and administrative workflows that reduce engineering and support burden.
CONFIDENCE: High.
FALSIFIER: The company needs only a few tightly controlled integrations and already employs deep identity expertise.

CLAIM: Buying reduces operational risk but does not outsource accountability.
EVIDENCE: The vendor can handle protocol correctness and credential infrastructure, while the SaaS must still securely implement authorization, tenant boundaries, provisioning, webhook verification, recovery policies, and failure behavior.
CONFIDENCE: High.
FALSIFIER: None; responsibility remains shared under either option.

CLAIM: Vendor cost and lock-in are usually less dangerous here than understated build costs.
EVIDENCE: Self-hosting adds maintenance, security patching, abuse prevention, uptime, migrations, compliance evidence, and on-call ownership. Standard protocols and an internal identity abstraction can constrain vendor dependency.
CONFIDENCE: Medium-high.
FALSIFIER: A realistic three-year model shows vendor pricing materially damages unit economics while the team can operate an equivalent system with lower fully loaded cost and risk.

VERDICT: A — Biggest risk: vendor pricing/lock-in becoming punitive as users and enterprise connections grow. Cheapest discriminating test: obtain binding price estimates for the next three years using projected MAU, SSO connections, MFA, and support needs; choose B only if the vendor case materially breaks unit economics and a named internal owner can demonstrate production-ready SAML, MFA recovery, patching, audit, and on-call capability.
