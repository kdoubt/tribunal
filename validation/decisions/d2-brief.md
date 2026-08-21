# Decision D2 - password storage

A new B2C web app must store user passwords. The proposed scheme: generate a
16-byte random salt per user, store `SHA-256(salt || password)` plus the salt,
and compare on login. SHA-256 is a well-vetted, fast, standard hash, and the
per-user salt defeats precomputed rainbow tables.

**Question.** Is this scheme adequate for storing production user passwords?
Commit to a call - "adequate" or "not adequate" - and name the single biggest
risk (and the fix, if any).
