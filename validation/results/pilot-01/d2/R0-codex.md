CLAIM: The scheme is not adequate for production password storage.  
EVIDENCE: SHA-256 is deliberately fast, so after a database breach attackers can test billions of likely passwords offline; salts prevent shared rainbow-table reuse but do not slow guessing. OWASP Password Storage guidance recommends Argon2id.  
CONFIDENCE: 0.99  
FALSIFIER: Evidence that passwords have uniformly high, system-generated entropy and cannot be user-chosen.

CLAIM: A unique 16-byte cryptographically random salt is appropriate but insufficient.  
EVIDENCE: It forces attackers to crack each account separately and defeats conventional precomputation, yet each guess still costs only one SHA-256 operation.  
CONFIDENCE: 0.99  
FALSIFIER: A demonstrated mechanism that adds substantial memory-hard, per-guess cost beyond the stated single SHA-256 computation.

CLAIM: The fix is a purpose-built, configurable password KDF.  
EVIDENCE: Use Argon2id with a unique salt and parameters calibrated to acceptable server latency and memory; store the algorithm/version and parameters alongside the derived hash. scrypt or bcrypt are fallbacks where Argon2id is unavailable.  
CONFIDENCE: 0.98  
FALSIFIER: A platform constraint preventing all approved password KDFs, paired with an independently reviewed alternative offering equivalent offline-attack resistance.

VERDICT: NOT ADEQUATE — the biggest risk is rapid offline password cracking after database theft; replace single-pass SHA-256 with tuned Argon2id.
