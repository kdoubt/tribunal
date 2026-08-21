# Claim Ledger - api-auth-jwt-vs-sessions

The orchestrator's working state after Round 0. Both seats answered in
isolation; their claims are deduplicated here by substance. A claim
independently made by **both** seats is `agreed-r0` (the only status that may
be reported as panel consensus). A decision-relevant claim only one seat
raised, uncontested by the other, stays `open` (reported, not endorsed).

| id | author | claim - one sentence, verbatim from a seat (for `agreed-r0` rows, one seat's phrasing of the substance both stated independently; full R0 in the round0 files) | evidence | provenance | confidence | status | decision-relevant |
|----|--------|-----|----|----|----|----|----|
| A1 | A, B | Pure option A cannot satisfy the MUST of logout or "kill this account now" taking effect within seconds, because a signed JWT remains valid until `exp` at every verifier that only checks signature and claims. | RFC 7009 §5; OWASP ASVS 7.4.1; MDN Session Management | PANELIST-CLAIM (EXTERNAL) | 0.99 / 0.93 | **agreed-r0** | yes |
| A2 | A, B | The usual patches that make A meet 7.4.1 (jti denylist, per-user token version, per-user key rotation) put a shared store back on the hot path, so "stateless JWT + revocation" is B's operational cost plus JWT's extra failure modes rather than a simpler alternative. | OWASP JWT Cheat Sheet (revocation); RFC 8725 | PANELIST-CLAIM (EXTERNAL) | 0.96 / 0.88 | **agreed-r0** | yes |
| A3 | A, B | Multi-tenant isolation is safer when tenant membership is an attribute of the server-side session record (loaded as source of truth) than when it is a client-held JWT claim that every one of ~50 services must remember to enforce. | Mechanism (opaque lookup returns bound tenant); RFC 8725 §3.8-3.12 (confused-deputy) | PANELIST-CLAIM (EXTERNAL) | 0.98 / 0.80 | **agreed-r0** | yes |
| A4 | A, B | Use the same opaque-session model with different transports: `Secure`, `HttpOnly`, appropriately `SameSite` cookies plus CSRF protection for the SPA, and an `Authorization` header stored in platform-secure storage for mobile. | OWASP Session Management Cheat Sheet; MDN | PANELIST-CLAIM (EXTERNAL) | 0.95 / 0.84 | **agreed-r0** | yes |
| A5 | A, B | Horizontal scale across ~50 gateway-fronted services does not overturn (1)–(3): the gateway can validate the opaque ID against one replicated Redis/DB on each request (O(1)), then inject identity inward; that is ordinary session-store scale, not a reason to accept unrevocable bearer JWTs. | Mechanism (gateway + Redis GET); MDN (prefer centralized) | PANELIST-CLAIM (EXTERNAL) | 0.94 / 0.81 | **agreed-r0** | yes |
| B1 | B | On blast radius, a JWT signing secret is a tenant-wide minting primitive: compromise (or `alg` confusion / `none`) lets an attacker forge tokens for any tenant until keys rotate, while rotating a global key either mass-invalidates everyone or leaves a dual-key window; an opaque session ID is unforgeable without store write access and dies when its row is deleted. | RFC 8725 / BCP 225 §2-3 (`alg` threats); OWASP (≥64-bit CSPRNG IDs) | PANELIST-CLAIM (EXTERNAL) | 0.86 | open (uncontested; A raised the same key/config risk in passing) | yes |
| A6 | A | Maintain subject-to-session and tenant-to-session indexes so "kill this account" can revoke every active device atomically or within a measured seconds-bounded propagation window. | OWASP Session Management (server-side invalidation + logging) | PANELIST-CLAIM (EXTERNAL) | 0.97 | open (uncontested; a mechanism detail of the agreed B design) | yes |

## Dispute rule outcome

**No disputed claims.** Every decision-relevant claim is either `agreed-r0`
(both seats, independently) or `open` (one seat's uncontested mechanism detail
of the same agreed design). There is nothing for Round 1 to cross-examine.

## Stop

Stop rule **(a): Round 0 already agreed** on everything decision-relevant.
Round 1 is not run - relaying non-disputed claims would only invite
politeness convergence on a conclusion already reached in isolation.
