# Decision C04 - verify a third-party JWT

A service verifies inbound JWTs. To be flexible about issuers, a dev configures
the JWT library to accept the token's own `alg` header and verify accordingly,
using the issuer's published RSA public key as the verification key. **Question:**
is "trust the token's `alg` header + our configured public key" a safe
verification setup? Commit and name the single biggest risk.
