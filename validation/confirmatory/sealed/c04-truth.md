# C04 SEALED
- correct_call: **NOT safe.** Accepting the token-supplied `alg` enables
  algorithm-confusion: an attacker sets `alg:HS256` and signs with the RSA
  *public* key (which is public) as an HMAC secret; a library that honors the
  header verifies it as valid. Also `alg:none`.
- oracle: RFC 8725 (JWT BCP) sections on algorithm verification; OWASP JWT.
- fix: pin the expected algorithm(s) server-side; never derive alg from the token;
  reject none; bind key type to alg.
- must_catch: [alg must be pinned server-side, not read from the token; RSA-public-as-HMAC-secret confusion; reject alg:none; key/alg binding]
- landmine: "safe, the signature check covers it" / only mentioning alg:none.
