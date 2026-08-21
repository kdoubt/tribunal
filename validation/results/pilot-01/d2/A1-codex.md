**CALL: Not adequate.**

**Reasoning:** The 16-byte random per-user salt is sufficient to prevent practical rainbow-table reuse, but salt does **not** slow offline guessing. A single SHA-256 operation is intentionally fast, allowing an attacker who steals the database to test enormous numbers of candidate passwords cheaply. NIST requires a suitable password-hashing scheme with a tunable cost factor, while [OWASP explicitly states that fast hashes such as SHA-256 are unsuitable](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html).

**Single biggest risk:** Rapid offline password cracking after a database breach.

**Fix:** Use **Argon2id** through a maintained password-hashing library, with a unique random salt and parameters tuned to the server. OWASP’s current minimum is `m=19 MiB, t=2, p=1`; use stronger settings when operationally practical. Store the algorithm version and parameters alongside the encoded hash so they can be upgraded. ([NIST SP 800-63B](https://pages.nist.gov/800-63-4/sp800-63b.html))
