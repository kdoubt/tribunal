**CLAIM:** No. The token’s `alg` is attacker-controlled metadata, not a trusted policy decision.  
**EVIDENCE:** JWT BCP requires callers to specify an allowed algorithm set and says each key must be bound to exactly one algorithm. [RFC 8725 §3.1](https://www.rfc-editor.org/rfc/rfc8725.html#section-3.1)  
**CONFIDENCE:** Very high.  
**FALSIFIER:** Configuration independently pins the expected algorithm per issuer/key and rejects every other `alg` before verification.

**CLAIM:** The single biggest risk is **algorithm/key confusion**, especially `RS256 → HS256`.  
**EVIDENCE:** An attacker changes `alg` to `HS256` and signs a forged token using the publicly available RSA public key bytes as the HMAC secret. Vulnerable libraries then accept it. This exact attack is documented in [RFC 8725 §2.1](https://www.rfc-editor.org/rfc/rfc8725.html#section-2.1).  
**CONFIDENCE:** Very high, though exploitability depends on the library/version and key parsing.  
**FALSIFIER:** The library categorically rejects symmetric algorithms when supplied an RSA public key and the application allowlists only the intended RSA algorithm.

**CLAIM:** A published RSA key authenticates nothing unless it is securely bound to a trusted issuer and intended algorithm.  
**EVIDENCE:** RFC 8725 requires validating that the cryptographic key belongs to the claimed issuer; issuer and audience must also be validated. [RFC 8725 §§3.8–3.9](https://www.rfc-editor.org/rfc/rfc8725.html#section-3.8)  
**CONFIDENCE:** High.  
**FALSIFIER:** Trusted issuer configuration—not token-controlled data—selects a pinned JWKS/key, algorithm, and expected audience.

**VERDICT:** **REJECT — unsafe configuration.** Biggest risk: attacker-chosen algorithm causes RSA/HMAC key confusion and forged-token acceptance. Cheapest discriminating test: submit a forged `HS256` JWT signed with the published RSA public-key bytes; it must be rejected, then enforce an explicit per-issuer allowlist such as exactly `RS256`.
