## Decision memo — C04

**CALL: No—reject this setup; pin an allowlist of algorithms per trusted issuer/key and treat the token’s `alg` only as a value to validate against that policy.**

**Single biggest risk:** algorithm confusion, especially **RS256 → HS256**. An attacker changes `alg` to `HS256`, signs arbitrary claims using the publicly available RSA public key as the HMAC secret, and a vulnerable or misconfigured verifier accepts the forged token. Variants include `alg: none` or substitution to another permitted algorithm. This is an authentication and authorization bypass, not merely cryptographic weakening.

Key risks:

- The attacker controls the JWT header; it cannot select verification policy.
- One public key accepted across algorithm families violates JWT best practice: each key must be bound to exactly one algorithm.
- Library protections vary by version and key representation; recent vulnerabilities show that relying on automatic key-type detection is unsafe.
- Flexible issuers also require explicit issuer-to-key/algorithm mapping plus validation of `iss`, `aud`, expiry, and key provenance.

The IETF JWT Best Current Practices requires callers to specify the supported algorithm set, forbids other algorithms, and requires each key to be used with exactly one algorithm. It explicitly documents RS256-to-HS256 confusion. [RFC 8725 §§2.1, 3.1](https://www.rfc-editor.org/rfc/rfc8725.html#section-3.1)

**Cheapest decisive test:** create a JWT with `alg: HS256`, attacker-controlled privileged claims, and an HMAC computed using the issuer’s published RSA public-key bytes. Send it through the production verification configuration.

**Wrong-choice criterion:** if that token—or any token whose algorithm is not the issuer’s explicitly configured algorithm—passes signature verification, the design is compromised. Keep this as a regression test and require fail-closed rejection before claim processing.
