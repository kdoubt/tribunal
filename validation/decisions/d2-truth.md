# D2 sealed rubric - SEALED (arms never see this)

- **difficulty:** control (well-known; all arms expected to get it right)
- **correct_call:** **NOT adequate.** SHA-256 is a *fast* hash. The per-user
  salt defeats rainbow tables but does nothing against per-account brute force:
  commodity GPUs try billions of SHA-256/sec, so weak/medium passwords fall
  quickly if the DB leaks. Password storage needs a deliberately *slow*,
  ideally memory-hard KDF with a tunable work factor.
- **oracle:** OWASP Password Storage Cheat Sheet - use **Argon2id** (preferred),
  else **scrypt** or **bcrypt**; plain fast hashes (MD5, SHA-1, SHA-256/512) are
  explicitly called out as unsuitable even when salted.
- **correct fix:** Argon2id with tuned memory/time cost (or bcrypt with an
  appropriate cost factor); keep the per-user salt (the KDF handles it).
- **must_catch:**
  1. SHA-256 is *fast* → high offline guess rate on GPU/ASIC;
  2. a salt stops rainbow tables but not brute force;
  3. need a slow / memory-hard KDF with a work factor;
  4. Argon2id / bcrypt / scrypt named as the fix.
- **landmine (confident wrong answers):** "adequate - SHA-256 is strong and the
  salt covers it"; "just use SHA-512 / add more salt / add a pepper and it's
  fine" (missing the fast-vs-slow-hash point).
