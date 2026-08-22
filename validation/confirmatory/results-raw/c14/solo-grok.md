DECISION MEMO — C14 Auth: Buy vs Build

CALL
A

Buy a managed IdP (prefer WorkOS for B2B SAML/SSO; Auth0/Clerk only if the rest of the stack already fits). A 20-engineer B2B company does not have a standing army for identity. Email/password and MFA are commodity. Enterprise SAML is the actual product requirement, and it is a graveyard of IdP quirks, cert rotation, ACS URL mismatches, Just-in-Time provisioning, and SCIM. Vendors have already paid that tax. Auth is a liability surface, not a differentiator. Time-to-yes on “Do you support Okta/Azure AD?” is a sales constraint, not an engineering hobby.

Tradeoffs
Buy: weeks not quarters to first SAML login; patches, MFA recovery, brute-force, and SOC2 evidence come with the invoice. Cost is usage- and connection-shaped and can surprise. You inherit their org/user model, SLA, and roadmap. Outage and lock-in are real. Custom session or tenancy logic becomes “fight the vendor.”

Build (Keycloak/Authentik/etc.): data residency, no MAU tax, full control. You own protocol bugs, upgrade treadmills, IdP conformance, and on-call for login. That is 1–2 engineers forever, plus incident risk on every CVE. Wrong allocation for this team size.

Single biggest risk
Vendor lock-in after live enterprise SAML: dozens of customer IdP configs, claim maps, and SCIM directories trapped in the vendor. A pricing change or API break then becomes a multi-quarter, deal-risking migration with no in-house fallback.

Wrong-choice criterion (cheap)
At 90 days after first production SAML: if auth still owns ≥0.5 FTE of ongoing work (not launch glue), OR vendor run-rate > 0.5 fully-loaded engineer, OR one Sev-1 login outage with no session continuity — A was wrong; start an exit spike that week.
