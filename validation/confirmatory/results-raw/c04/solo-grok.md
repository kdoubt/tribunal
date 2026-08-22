# DECISION MEMO — C04: Trust the token `alg` + RSA public key

**CALL:** No. Do not trust the token’s `alg`. Pin allowed algorithms (e.g. `RS256` only) and key type; reject anything else.

## Why

A JWT’s `alg` is attacker-controlled. Verification must be: *this key, this algorithm, this issuer* — not “whatever the token asked for, using the key we already have.” Flexible-issuer setups still pin `alg` (and `iss`/`aud`/`kid`) per trusted issuer; they do not let the token pick the crypto.

## Key risks

- **Algorithm confusion (RS→HS):** Attacker sets `alg: HS256` and HMAC-signs with the published RSA public key as the secret. Libraries that “verify according to `alg`” treat the RSA pubkey as an HMAC key. Token verifies. Attacker is now any user.
- **`alg: none`:** Unsigned token accepted if the library honors `alg`.
- **Cross-algorithm / key-type mixups:** `ES256` vs `RS256`, PKCS#1 vs raw key bytes — same class of bug: header chooses a primitive the configured key was never meant for.
- **Issuer flexibility as cover:** “We don’t know the alg until we see the token” is how this gets shipped. Per-issuer allowlists exist for a reason.

## Single biggest risk

**RSA/HMAC algorithm confusion.** One forged token, signed with a public key you already publish, becomes a valid session. No key theft required.

## Cheapest test that you chose wrong

Take the issuer RSA public key (PEM/SPKI as the library would ingest it). Build a JWT with `alg: HS256` (and a plausible `iss`/`sub`/`exp`). HMAC-SHA256-sign it using that public-key material as the secret. Send it to the verifier.

- **Accepted** → the setup is broken. Ship a pin: allowlist `RS256` (or the one real alg), bind key type to RSA, reject `none`/`HS*`.
- Also try `alg: none` with no signature. If that passes, same verdict, even cheaper.

Do not “fix” this with “we only fetch RSA keys from JWKS.” JWKS does not stop a confused verifier from using that RSA key as HMAC. Pin `alg` in code, not in the token.
