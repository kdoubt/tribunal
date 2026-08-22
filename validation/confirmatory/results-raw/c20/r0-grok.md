**CALL: No — ordinary `==` is not adequate. Use a constant-time MAC compare.**

**CLAIM 1:** `received == expected` on hex HMAC strings is variable-time.
**EVIDENCE:** Equality in C/Python/Java/Go typically exits at the first differing byte (or equivalent). HMAC-SHA256 hex is 64 chars; a remote or co-located observer who can distinguish “mismatch at nibble *i*” vs “*i+1*” learns the expected MAC prefix. Hex leaks ~4 bits per step (16 guesses/nibble), coarser than raw-byte compare.
**CONFIDENCE:** 0.9
**FALSIFIER:** This runtime’s string `==` is documented/constant-time for equal-length ASCII, *and* both sides are length-checked before compare.

**CLAIM 2:** A recovered expected MAC forges the webhook trust boundary.
**EVIDENCE:** Expected value is `hex(HMAC-SHA256(secret, body))`. For an attacker-chosen body it is fixed. Once the full hex is known, `received` matches and the receiver accepts a fabricated event (payments, user changes, etc.). Replay-of-captured-traffic is a weaker cousin; this is injection of *new* events.
**CONFIDENCE:** 0.85
**FALSIFIER:** The MAC input includes a server-only nonce the attacker cannot set, or the receiver rejects any body the attacker can time.

**CLAIM 3:** WAN exploit is hard; that does not make `==` adequate.
**EVIDENCE:** HTTP/TLS jitter usually swamps memcmp nanoseconds; public webhook timing breaks are rare. Co-tenant, debug, or localhost measurement is the realistic channel. `hmac.compare_digest` / `crypto.timingSafeEqual` / `subtle.constantTimeCompare` cost is negligible. Crypto hygiene treats “known oracle class + free fix” as not adequate.
**CONFIDENCE:** 0.8
**FALSIFIER:** A published remote break of this exact `==`/hex/HMAC-SHA256 stack over the public internet, *or* a requirement that only WAN attackers exist and local timing is in-scope-impossible.

**CLAIM 4:** Hex `==` adds correctness footguns besides timing.
**EVIDENCE:** Case (`A` vs `a`), missing `0x`, truncated headers, and str-vs-bytes compare fail closed or open depending on language. Constant-time APIs force equal-length byte digests and fail closed on length mismatch.
**CONFIDENCE:** 0.75
**FALSIFIER:** Canonicalization to lowercase raw bytes of fixed size is already proven before `==`.

**VERDICT:** **Do not ship `==`; constant-time compare the raw digests.** Biggest risk: timing oracle yields a valid HMAC for an attacker body → forged webhooks. Cheapest test: same-host loop of 64-char hex with shared prefix length 0..64; if mean latency trends with prefix, the oracle is real — then swap to `compare_digest` and re-run (trend must vanish).
