**CALL: No — unjittered exponential backoff is not sufficient.**

**CLAIM 1:** Simultaneous 503s keep clients phase-locked; 1s/2s/4s/8s/16s produces retry *waves*, not a smooth drain.
**EVIDENCE:** Shared outage starts every client’s timer at the same instant. Without jitter, attempt *k* lands in a tight band around \(T+\sum 2^{i}\). AWS “Exponential Backoff And Jitter” and the SRE retry-storm literature treat this as the default failure mode of this exact policy.
**CONFIDENCE:** 0.92
**FALSIFIER:** Packet captures or client traces during a fleet-wide 503 show retry timestamps uniformly spread inside each backoff bucket (std-dev comparable to the interval), not clustered.

**CLAIM 2:** The first wave (all clients at ~T+1s) is the one that re-breaks a recovering downstream.
**EVIDENCE:** Brief outages (sub-second to a few seconds) often clear *before* the 1s retry. A healthy-but-cold instance then eats N extra requests at once. Capacity is lowest at recovery (connection setup, cache miss, thread warmup). One synchronized spike can 503 again and reset every client’s backoff.
**CONFIDENCE:** 0.88
**FALSIFIER:** Inject a 500ms 503 for all clients against a service at ~70% of SLO capacity; RPS at T+1s stays within normal variance and error rate does not re-spike.

**CLAIM 3:** A 5-try cap still multiplies offered load up to ~5× for the duration of the incident; backoff only *spaces* the extra load, it does not *budget* it.
**EVIDENCE:** Each failed call is replaced by up to 5 more. Hundreds of clients × 5 = a large multiple of baseline. No retry budget, no circuit breaker, no `Retry-After` means the downstream cannot refuse the herd except by failing again.
**CONFIDENCE:** 0.85
**FALSIFIER:** During a multi-second 503, measured client-originated RPS never exceeds ~1.2× baseline (clients abort, shed, or honor a budget).

**CLAIM 4:** Adding jitter later does not make *this* policy sufficient; the described design also lacks decorrelated jitter *and* a fleet-wide retry cap.
**EVIDENCE:** Full/equal jitter breaks sync but still lets every client retry. At high fan-in, independent retries remain a DDoS of the dependency. Decorrelated jitter + per-client retry budget (e.g. 10% of traffic) is the usual fix.
**CONFIDENCE:** 0.80
**FALSIFIER:** With only full jitter added (no budget), a 10s total outage still keeps downstream CPU/queue under the crash threshold.

---

**VERDICT:** **Insufficient — do not ship this as the shared-service retry policy.**  
**Biggest risk:** Synchronized T+1s retry storm that knocks a recovering service back into 503s (retry feedback loop).  
**Cheapest test:** Chaos: force 503 for all clients for 800ms, plot inbound RPS in 100ms buckets around T+1s / T+3s / T+7s. If you see spikes ≫ baseline, the policy fails; rerun with full jitter + a 10% retry budget as the control.
