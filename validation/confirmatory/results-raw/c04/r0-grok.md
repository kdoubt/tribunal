**UNSAFE. Do not ship.**

### CLAIM 1
**CLAIM:** Trusting the token’s `alg` while verifying with a configured RSA public key is an algorithm-confusion hole. An attacker sets `alg` to `HS256`/`HS384`/`HS512` and HMAC-signs with the *public* RSA key as the secret. The verifier then “succeeds.”
**EVIDENCE:** JWT `alg` is attacker-controlled. Many libraries treat “verify with this key” as “use whatever `alg` says,” so an RSA key object is reused as an HMAC secret. This is the 2015 McLean class of bugs (jose/jwt libraries, CVE-2015-9235 and cousins). The public key is public, so forgery needs no private key.
**CONFIDENCE:** 95
**FALSIFIER:** Verifier *hard-rejects* HMAC algs when the configured key is RSA/EC (key-type pin), or HMAC-with-pubkey tokens fail closed in the actual library.

### CLAIM 2
**CLAIM:** “Flexible issuers” does not require trusting `alg`. Pin algorithm *and* key per issuer (JWKS `kty`/`alg`/`use`/`kid`). The header may *select which key*, never *which crypto primitive*.
**EVIDENCE:** RFC 7515/7517: receivers must know expected algorithms. JWKS already carries `alg`/`kty`. Cross-issuer flexibility is a key set, not an open `alg` switch.
**CONFIDENCE:** 90
**FALSIFIER:** A documented, key-type-enforcing API that still reads `alg` but cannot apply HMAC to an RSA key.

### CLAIM 3
**CLAIM:** `alg: none` is a secondary, library-dependent skip-verify path; same root cause (header drives policy).
**EVIDENCE:** Historical `none` acceptance when algorithm is taken from the token. Less common in current libs, still the same control-plane error.
**CONFIDENCE:** 70
**FALSIFIER:** Library rejects `none` *and* non-asymmetric algs before any key op.

### CLAIM 4
**CLAIM:** Biggest production impact is **forged admin/session tokens** against any RSA-JWT consumer using this pattern—not “wrong issuer,” not key rotation.
**EVIDENCE:** One accepted HS256-with-pubkey token impersonates any subject the app trusts. No IdP compromise required.
**CONFIDENCE:** 88
**FALSIFIER:** App additionally binds tokens via sender-constrained keys (DPoP/mTLS) that a forged JWT cannot satisfy.

---

**VERDICT:** **CALL: reject this setup.**  
**Biggest risk:** RS256→HS256 confusion — HMAC-forge with the published RSA public key.  
**Cheapest test:** Build a JWT `{"alg":"HS256"}` HMAC-signed with the PEM/SPKI bytes of the issuer RSA public key; if the service accepts it, you have a total auth bypass. Also try `alg: none`. Pin `RS256` (or the JWKS `alg`) and refuse HMAC for RSA keys.
