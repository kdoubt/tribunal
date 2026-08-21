# C20 SEALED
- correct_call: **NOT adequate.** Ordinary string `==` typically short-circuits on
  the first differing byte, leaking a **timing side-channel** an attacker can use
  to forge a valid signature byte-by-byte. Use a **constant-time** comparison
  (e.g. `hmac.compare_digest`, `crypto.timingSafeEqual`).
- oracle: OWASP; language docs for hmac.compare_digest / crypto.timingSafeEqual
  (both explicitly for timing-attack-resistant comparison).
- must_catch: [`==` short-circuits -> timing leak; attacker forges byte-by-byte; use constant-time compare (compare_digest/timingSafeEqual); compare raw bytes of equal length]
- landmine: "adequate, it's just comparing two strings" / no mention of timing.
