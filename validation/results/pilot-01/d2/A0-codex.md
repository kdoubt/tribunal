**CALL: Not adequate.**

The salt design is sound, but SHA-256 is intentionally fast. If the password database is stolen, attackers can test billions of guesses efficiently using GPUs/ASICs. Salts prevent rainbow-table reuse; they do not slow per-user brute-force or dictionary attacks.

**Single biggest risk:** Rapid offline cracking of weak or reused passwords after a database breach.

**Fix:** Use a purpose-built, memory-hard password hashing function—prefer **Argon2id** with a unique 16-byte salt and parameters tuned to the production environment (targeting roughly 100–500 ms per verification). Store the algorithm and parameters with each hash to support future upgrades. If Argon2id is unavailable, use scrypt; bcrypt or PBKDF2 are compatibility fallbacks.
