**CALL:** Ordinary `==` is not adequate — compare HMAC-SHA256 signatures with a constant-time equality check (e.g. `hmac.compare_digest` / `crypto.timingSafeEqual`) over equal-length byte strings.

**Why.** Python/JS/Go `==` on strings/bytes short-circuits on the first mismatch. That turns the header comparison into an oracle: each extra correct hex nibble (or byte) of a forged signature costs a slightly longer compare. Over enough samples an attacker reconstructs a valid HMAC and forges webhooks (replay, fake events, account takeover — whatever this receiver trusts).

HMAC-SHA256 itself is not the defense. A correct digest compared unsafely is still a leaky secret. Length-normalize first (reject mismatched sizes in constant time or hash both sides then compare the hashes); never `==` the hex strings.

**Key risks**
- Timing oracle on the expected signature → remote forgery of trusted events.
- Encoding mismatch (hex vs raw, `sha256=` prefix, Unicode) causing false rejects — or worse, a fast-fail path that *amplifies* the leak.
- “Network jitter makes this unexploitable” as an excuse: jitter raises sample count; it does not close the channel. Standards and reviewers will fail you anyway.

**Single biggest risk:** Remote reconstruction of the valid HMAC via short-circuiting `==`, then forged webhook payloads accepted as authentic.

**Cheapest falsifier:** From off-box, send two large equal-size batches against a known body: (A) signature wrong at nibble 0, (B) prefix-correct for increasing nibble depth. If median/trimmed-mean latency rises with prefix length (even tens of µs after jitter averaging), `==` is leaking and the choice is wrong. If you cannot even run that test because compare is not constant-time in source, treat that as already failed.
