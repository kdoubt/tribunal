# Round 0 - Seat A (Codex CLI, GPT-5-class) - independent position

> Verbatim seat output. Isolated: this seat did not see Seat B's answer.

Plausible framings: OAuth token-format choice; session-state architecture; or a revocation-SLA decision. I choose the revocation-SLA framing because “within seconds” is a hard constraint that eliminates designs whose validity persists solely through cryptographic expiry.

CLAIM: Choose B: opaque server-side sessions validated centrally at the gateway on every request.
EVIDENCE: A random opaque ID can be invalidated by deleting or marking its store record revoked, so the next request fails; stateless JWT verification has no equivalent revocation input. EXTERNAL: [RFC 7009 §5](https://www.rfc-editor.org/rfc/rfc7009.html#section-5) warns that access tokens are not immediately invalidated when access-token revocation is unsupported.
CONFIDENCE: 0.99
FALSIFIER: A demonstrates sub-second account-wide access-token revocation without consulting mutable server-side state during request validation.

CLAIM: Store both `tenant_id` and `subject_id` in the authoritative session record and require the request’s resolved tenant to equal that tenant before authorization.
EVIDENCE: This server-side equality check prevents a valid session ID from being replayed under another tenant; relying on a caller-supplied tenant header or unverified token claim creates a confused-deputy/cross-tenant authorization failure.
CONFIDENCE: 0.98
FALSIFIER: The product intentionally permits every authenticated identity to exercise identical authority across all tenants.

CLAIM: Maintain subject-to-session and tenant-to-session indexes so “kill this account” can revoke every active device atomically or within a measured seconds-bounded propagation window.
EVIDENCE: Deleting only the presented session implements logout-one-device, not account-compromise containment; an indexed generation counter or transactional bulk revocation makes all prior sessions fail their next lookup. EXTERNAL: [OWASP Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) requires server-side invalidation and lifecycle logging.
CONFIDENCE: 0.97
FALSIFIER: A load test plus failure test shows account-wide revocation routinely exceeds the required SLA despite correctly indexed records.

CLAIM: Put session validation at the gateway and deny requests if the session store or revocation check is unavailable.
EVIDENCE: Central validation avoids implementing security-critical token parsing across roughly 50 services, while fail-closed behavior prevents a Redis outage from silently disabling revocation; the concrete tradeoff is that the gateway/store becomes an availability dependency.
CONFIDENCE: 0.94
FALSIFIER: Services remain directly reachable around the gateway, or the required availability target cannot tolerate a replicated session-store dependency.

CLAIM: Use the same opaque-session model with different transports: `Secure`, `HttpOnly`, appropriately `SameSite` cookies plus CSRF protection for the SPA, and an `Authorization` header stored in platform-secure storage for mobile.
EVIDENCE: Token format need not equal transport; HttpOnly reduces JavaScript token exposure, while cookie authentication requires CSRF defenses because browsers attach cookies automatically. EXTERNAL: [OWASP Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) specifies Secure/HttpOnly/SameSite protections and server-side termination.
CONFIDENCE: 0.95
FALSIFIER: The SPA cannot use first-party cookies or its deployment topology makes the selected cookie and CSRF controls unworkable.

CLAIM: Do not add self-contained external JWT access tokens initially; if internal signed assertions are later needed, make them gateway-only and request-bound rather than reusable bearer credentials.
EVIDENCE: Reusable JWTs reintroduce a post-revocation validity window unless every service consults a denylist/version store, which converts “stateless” A back into stateful validation and expands key, claim, issuer, and audience configuration failure modes. EXTERNAL: [RFC 8725 §§3.8–3.9](https://www.rfc-editor.org/rfc/rfc8725.html#section-3.8) mandates issuer and audience validation for JWTs.
CONFIDENCE: 0.96
FALSIFIER: Independent service authentication becomes mandatory and a proven centralized revocation mechanism preserves the seconds-level SLA without materially increasing operational or configuration risk.

VERDICT INPUT: **B — gateway-validated opaque server-side sessions backed by a replicated shared store; biggest risk: the session store becomes a security-critical availability dependency, especially if any failure path accidentally fails open.**
