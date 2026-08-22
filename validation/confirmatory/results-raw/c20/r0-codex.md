**CLAIM:** Ordinary `==` is not adequate for verifying webhook HMACs.  
**EVIDENCE:** Typical string equality may return after the first differing character, making runtime depend on the matching prefix. Cryptographic guidance calls for constant-time comparison of authentication tags.  
**CONFIDENCE:** High.  
**FALSIFIER:** Proof that the specific runtime’s `==` implementation is constant-time for these strings across all deployed versions and configurations.

**CLAIM:** Compare decoded MAC bytes—or strictly validated, normalized hex—using a constant-time primitive such as `hmac.compare_digest`, `crypto.timingSafeEqual`, or the platform equivalent.  
**EVIDENCE:** These APIs are specifically designed to avoid content-dependent comparison timing. Byte comparison also avoids case and representation ambiguity in hex strings.  
**CONFIDENCE:** High.  
**FALSIFIER:** The platform documents an equally strong constant-time guarantee for ordinary string equality.

**CLAIM:** Validate encoding and exact length before comparison, while returning indistinguishable failures.  
**EVIDENCE:** Constant-time APIs may reject unequal lengths, and permissive hex parsing can admit malformed or alternate representations. Length checks may reveal length, but MAC length is public and fixed.  
**CONFIDENCE:** High.  
**FALSIFIER:** The framework already guarantees a canonical, fixed-length header and uniform failure handling.

**VERDICT:** **CALL: No—replace `==` with a constant-time comparison.** Biggest risk: a timing side channel that reveals matching signature prefixes and could ultimately enable webhook forgery. Cheapest discriminating test: inspect the deployed equality implementation or benchmark many chosen signatures grouped by correct-prefix length; regardless of observed network noise, switching to the standard constant-time API is the safer, trivial fix.
