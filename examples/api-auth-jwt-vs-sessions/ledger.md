# Claim Ledger - api-auth-jwt-vs-sessions

The orchestrator's working state after Round 0. Both seats answered in
isolation; their claims are deduplicated here by substance. A claim
independently made by **both** seats is `agreed-r0` (the only status that may
be reported as panel consensus). A decision-relevant claim only one seat
raised, uncontested by the other, stays `open` (reported, not endorsed).

| id | author | claim (compressed - full text in the round0 files) | evidence | provenance | confidence | status | decision-relevant |
|----|--------|-----|----|----|----|----|----|
| A1 | A, B | Pure stateless JWT (signature + `exp` only) **cannot** meet the immediate-revocation MUST: a captured token stays valid at every verifier until expiry. | RFC 7009 §5; OWASP ASVS 7.4.1; MDN Session Management | PANELIST-CLAIM (EXTERNAL) | 0.99 / 0.93 | **agreed-r0** | yes |
| A2 | A, B | The patches that make JWT revocable (denylist / per-user token version / key rotation) put a shared store back on the hot path → that is B's cost *plus* JWT's extra failure modes, not a simpler design. | OWASP JWT Cheat Sheet (revocation); RFC 8725 | PANELIST-CLAIM (EXTERNAL) | 0.96 / 0.88 | **agreed-r0** | yes |
| A3 | A, B | Multi-tenant isolation is safer when `tenant_id` is bound in the **server-side record** (loaded as source of truth) than as a client-held JWT claim ~50 services must each remember to enforce. | Mechanism (opaque lookup returns bound tenant); RFC 8725 §3.8-3.12 (confused-deputy) | PANELIST-CLAIM (EXTERNAL) | 0.98 / 0.80 | **agreed-r0** | yes |
| A4 | A, B | One credential, two transports: `Secure`/`HttpOnly`/`SameSite` (`__Host-`) cookie + CSRF token for the SPA; the same opaque ID as an `Authorization` header for mobile. Transport ≠ token format. | OWASP Session Management Cheat Sheet; MDN | PANELIST-CLAIM (EXTERNAL) | 0.95 / 0.84 | **agreed-r0** | yes |
| A5 | A, B | Horizontal scale to ~50 services does not favor JWT: validate the opaque ID once at the gateway against one replicated store (O(1), ~1ms), then inject identity inward. "Stateless for scale" only wins if the revocation store is avoided - which the brief forbids. | Mechanism (gateway + Redis GET); MDN (prefer centralized) | PANELIST-CLAIM (EXTERNAL) | 0.94 / 0.81 | **agreed-r0** | yes |
| B1 | B | Blast radius: a JWT signing key is a **tenant-wide minting primitive** (one leaked HS256 secret or an `alg:none`/confusion bug forges tokens for *any* tenant until rotation); an opaque ID is unforgeable without store write access. | RFC 8725 / BCP 225 §2-3 (`alg` threats); OWASP (≥64-bit CSPRNG IDs) | PANELIST-CLAIM (EXTERNAL) | 0.86 | open (uncontested; A raised the same key/config risk in passing) | yes |
| A6 | A | "Kill this account now" needs subject→session and tenant→session **indexes** (or a per-user generation counter), so a compromise revokes every active device within the SLA, not just one. | OWASP Session Management (server-side invalidation + logging) | PANELIST-CLAIM (EXTERNAL) | 0.97 | open (uncontested; a mechanism detail of the agreed B design) | yes |

## Dispute rule outcome

**No disputed claims.** Every decision-relevant claim is either `agreed-r0`
(both seats, independently) or `open` (one seat's uncontested mechanism detail
of the same agreed design). There is nothing for Round 1 to cross-examine.

## Stop

Stop rule **(a): Round 0 already agreed** on everything decision-relevant.
Round 1 is not run - relaying non-disputed claims would only invite
politeness convergence on a conclusion already reached in isolation.
