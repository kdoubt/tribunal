**CALL:** No — unjittered shared exponential backoff will amplify a simultaneous 503 and can prevent recovery.

**Key risks**
- **Correlated retry waves.** Every client fails together, sleeps the same 1s / 2s / 4s / 8s / 16s, and hits the recovering service as one pulse. Backoff without jitter *concentrates* load, it does not spread it.
- **Retry amplification.** Five attempts ⇒ up to 5× offered load while the downstream is already unhealthy. A blip becomes a storm.
- **No client-side brake.** There is no retry budget, circuit breaker, or honor of `Retry-After`. Clients keep contributing after the outage is global and obvious.
- **Recovery collapse.** The first healthy second after the 503 is the most expensive: the t=1s wave can knock the service back down, which re-arms the next wave.

**Single biggest risk:** Synchronized retry waves (thundering herd). Identical schedules plus a common failure clock turn hundreds of clients into a metronome. That, not the exponential *shape*, is what makes the incident worse.

**Cheapest falsification:** In staging, freeze the downstream for ~2s (forced 503) with production-like client count; compare inbound RPS and error rate for 30s with retries on vs retries off (or vs the same policy plus full jitter). **You chose wrong if** recovery-window RPS exceeds ~1.2× steady-state, or you see discrete spikes at ~1s / 3s / 7s / 15s, or time-to-healthy is *longer* with retries than without. Fix: full jitter (or decorrelated jitter), a per-client retry budget (e.g. ≤10% of requests), and fail-fast / circuit-open on clustered 503s.
