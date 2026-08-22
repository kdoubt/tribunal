## Decision Memo — C06

**CALL:** No. Deterministic exponential backoff is insufficient; require exponential backoff with full jitter, a retry budget, and respect for `Retry-After`.

**Single biggest risk:** A synchronized retry storm (“thundering herd”). Because every client receives the 503 together and waits identical intervals, hundreds of clients retry together at approximately 1, 3, 7, 15, and 31 seconds. Backoff spaces the bursts apart; it does not smooth them. A recovering service can be overwhelmed repeatedly, extending a brief failure into a sustained outage. AWS explicitly warns that without jitter, simultaneous failures produce simultaneous retry bursts and recommends full jitter to distribute them. [AWS retry behavior](https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html)

**Key risks:**

- Up to five additional synchronized load waves during recovery.
- Retried traffic competes with new traffic, delaying stabilization.
- Retry layers in proxies, SDKs, and applications can multiply attempts.
- Retrying non-idempotent operations can duplicate side effects.
- Ignoring server-provided `Retry-After` discards explicit recovery guidance permitted with HTTP 503. [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html)

**Required policy:** Use full jitter: choose each delay uniformly from `0..min(cap, base × 2^attempt)`. Cap both attempts and total retry traffic; retry at one layer only.

**Cheapest decisive test:** In staging, have the downstream return 503 to 500 concurrent clients for two seconds, then restore it at constrained recovery capacity. Record retry arrivals in 100 ms buckets.

**Wrong-choice criterion:** If any post-recovery retry bucket exceeds the downstream’s safe request capacity—or recovery time/error count is worse than the same test using full jitter—the deterministic policy is rejected. Expect deterministic backoff to produce near-500-request spikes; full jitter should spread those requests across each backoff window.
