# Frozen Brief - auth for a new multi-tenant B2B API: stateless JWT vs server-side sessions

## Question under review

A greenfield multi-tenant B2B SaaS API (team of 3 engineers; will grow toward ~50 internal services behind a gateway; clients are a web SPA and a mobile app). Choose the authentication mechanism:
- **A) Stateless signed JWTs** - short-lived signed access tokens, no server session store; refresh tokens for renewal.
- **B) Opaque server-side sessions** - random session/token IDs backed by a shared store (e.g. Redis/DB), validated per request.

## Decision criteria (owner-supplied)

Optimize for, in order: (1) security - especially IMMEDIATE revocation on logout and on account compromise; (2) blast radius if we get it wrong; (3) operational simplicity for a 3-person team; (4) horizontal scalability across many services. If a hybrid clearly dominates, say so and name it precisely.

## Constraints

- MUST support immediate revocation (a logout or a "kill this account now" must take effect within seconds, not minutes).
- Multi-tenant; a token/session must never be usable across tenants.
- Mixed clients: browser SPA (cookie-capable) + mobile (custom header).
- Small team - avoid operationally heavy designs unless the security case demands it.

## Output contract for seats

Max 6 claims, each as:
CLAIM: <one sentence>
EVIDENCE: <a decisive pointer - a specific standard/BCP/OWASP guidance (EXTERNAL with source), a concrete mechanism, or a named failure mode; not a vibe>
CONFIDENCE: <0-1>
FALSIFIER: <the concrete observation that would prove this claim wrong>
Plus VERDICT INPUT - your one-line recommendation (A / B / a named hybrid) + the single biggest risk of your choice.
