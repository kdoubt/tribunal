DECISION MEMO — C14: Authentication

CALL

A

Buy a managed identity provider.

Authentication is security infrastructure, not product differentiation. For a 20-engineer B2B SaaS, managed identity is the fastest path to credible email/password auth, MFA, account recovery, session security, and especially enterprise SSO/SAML. Building on an open-source library avoids license fees but does not avoid ownership: protocol edge cases, tenant-specific SAML debugging, key rotation, abuse controls, vulnerability response, audit evidence, uptime, and support escalations remain yours.

Key tradeoffs and risks:

- Higher and potentially nonlinear cost as users, enterprise connections, or MFA usage grow.
- Vendor lock-in through provider-specific APIs, metadata, session semantics, and user records.
- Provider outages become login outages.
- Custom workflows and unusual authorization models may be constrained.
- Migration later will be painful, so keep application authorization and tenant membership outside the provider, use standard OIDC/SAML boundaries, and maintain an exportable user mapping.

The alternative’s apparent control is misleading. Self-hosting gives implementation control while creating a permanent security and operations burden that will consume senior engineering attention unpredictably. Enterprise SSO alone makes buying the better allocation of scarce staff.

Single biggest risk: vendor pricing power. Once customers, credentials, MFA enrollment, and enterprise SSO connections accumulate, switching costs let the provider raise effective prices or push required capabilities into expensive tiers.

Cheapest wrong-choice test: obtain a three-year quote using the expected user count, enterprise SSO connections, MFA usage, and growth case. If the provider’s annual cost reaches the fully loaded cost of one senior engineer—or gross-margin impact exceeds 2%—while the team can demonstrate a supported self-hosted alternative meeting the same security, uptime, recovery, audit, and SAML requirements, A was the wrong choice.
