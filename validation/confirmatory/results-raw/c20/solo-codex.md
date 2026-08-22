## Decision memo — C20

**CALL:** No—ordinary `==` is inadequate; compare with `hmac.compare_digest(received, expected)`.

**Single biggest risk:** A timing side channel can reveal how much of a candidate signature matches, potentially allowing an attacker with enough repeated requests and stable measurements to forge a valid webhook signature without knowing the secret.

**Key risks:**

- String equality may stop at the first differing character, making runtime depend on the matching prefix.
- Network noise makes exploitation harder, but does not remove the vulnerability; repeated measurements can amplify small differences.
- Rate limiting is useful defense-in-depth, not a substitute for constant-time comparison.
- Reject malformed signatures and require the expected fixed length/encoding. Length itself is not secret.

Python’s authoritative HMAC documentation explicitly recommends `compare_digest()` instead of `==` for externally supplied digests because it avoids content-based short-circuiting and reduces timing-attack exposure. [`hmac` documentation](https://docs.python.org/3/library/hmac.html)

```python
if not hmac.compare_digest(received, expected):
    reject_request()
```

A local 5-million-comparison check showed position-dependent `==` times: approximately 0.049 seconds for a first-character mismatch versus 0.055 seconds for a last-character mismatch. `compare_digest` remained approximately 0.18 seconds across mismatch positions. This is illustrative, not proof of remote exploitability.

**Cheapest wrong-decision test:** Send many otherwise identical requests whose supplied signatures differ at progressively later positions. If statistical analysis can distinguish any latency distribution by matching-prefix length, `==` has failed the security criterion. Since replacing it is a one-line, negligible-cost change already mandated by the library guidance, do not wait for that exploit test—use `compare_digest()` now.
