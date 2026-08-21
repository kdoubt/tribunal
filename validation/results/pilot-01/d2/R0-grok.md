I'll check current password-storage guidance so the call is grounded in named sources, not just general practice.**CLAIM.** `SHA-256(salt || password)` is not a password KDF: one SHA-256 invocation has no work factor and is designed to be cheap, so after a dump each guess is essentially free on GPU/ASIC.

**EVIDENCE.** SHA-256 is a general-purpose cryptographic hash (FIPS 180-4), not a password-based KDF. hashcat-class SHA-256 rates on a single high-end GPU are on the order of 10⁹–10¹⁰ guesses/s; a unique 16-byte salt does not change the per-guess cost. OWASP Password Storage Cheat Sheet (current) forbids fast hashes (MD5, SHA-1/256/512, SHA-3) for password storage and requires Argon2id (min ~19 MiB, t=2, p=1), else bcrypt / scrypt, else PBKDF2-HMAC-SHA-256 with a high iteration count when FIPS applies.

**CONFIDENCE.** 0.97

**FALSIFIER.** A measured result that one salted SHA-256 trial costs as much as Argon2id (or bcrypt cost ≥12 / PBKDF2 ≥600k HMAC-SHA-256) on the same attacker hardware.

---

**CLAIM.** The proposal’s salt argument is true but incomplete: a per-user salt kills rainbow tables and identical-password detection, and does **not** stop dictionary/brute-force of each hash after leakage.

**EVIDENCE.** OWASP Password Storage Cheat Sheet, “Salting”: unique salts force the attacker to crack hashes one at a time and block precomputed tables — they do not add a cost factor. LinkedIn 2012 (unsalted SHA-1) is the rainbow-table failure mode; subsequent salted-SHA dumps still fall to GPU dictionaries because user passwords have low entropy. B2C users do not supply 128-bit random secrets.

**CONFIDENCE.** 0.95

**FALSIFIER.** A B2C population whose passwords have enough entropy that a single SHA-256 evaluation is an adequate stretching function (e.g., generated 128-bit secrets, not user-chosen passwords).

---

**CLAIM.** The scheme is non-compliant with the two named baselines that define “adequate” for production verifiers.

**EVIDENCE.** NIST SP 800-63B §5.1.1.2: verifiers SHALL store memorized secrets in a form resistant to offline attacks; they SHALL be salted and hashed with a suitable one-way KDF that takes a password, a salt, **and a cost factor**, with a memory-hard function SHOULD. Single-iteration SHA-256 has no cost factor. OWASP’s ranked primitives all include an adaptive cost parameter; this scheme has none.

**CONFIDENCE.** 0.94

**FALSIFIER.** A current OWASP or NIST 800-63B revision that lists unsalted-or-salted SHA-256 (one iteration, no KDF wrapper) as an approved password-storage construction.

---

**CLAIM.** The 16-byte salt itself is fine; inadequacy is the primitive, not salt length. A pepper or constant-time compare would be extras, not the blocking issue.

**EVIDENCE.** NIST 800-63B requires salt ≥32 bits, randomly chosen; 128 bits exceeds that. Modern KDFs (Argon2id, bcrypt, PBKDF2) generate and embed salt. Pepper (secret server-side key, OWASP “consider”) and `hmac.compare_digest` are defense-in-depth. Replacing SHA-256 with Argon2id (or bcrypt / PBKDF2) at recommended parameters, keeping the 16-byte salt, is the sufficient fix.

**CONFIDENCE.** 0.90

**FALSIFIER.** A successful production attack whose root cause is 128-bit salt collision or missing pepper, while a correctly parameterized Argon2id/bcrypt/PBKDF2 hash of the same passwords would also have fallen.

---

**VERDICT.** **not adequate** — biggest risk is offline GPU cracking of a leaked hash table of user-chosen passwords; fix is Argon2id (bcrypt or PBKDF2-HMAC-SHA-256 if constrained), not a faster SHA.
