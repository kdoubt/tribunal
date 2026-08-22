CLAIM: Exponential backoff alone is **not sufficient**.

EVIDENCE: Because all clients fail simultaneously and use identical delays (1s, 2s, 4s…), their retries remain synchronized. The downstream receives large retry waves at predictable times rather than a smoothed load.

CONFIDENCE: Very high.

FALSIFIER: Client retry timers are already independently randomized enough to spread each retry across the backoff interval.

CLAIM: The single biggest risk is a synchronized retry storm (“thundering herd”).

EVIDENCE: Each failed recovery attempt can trigger another fleet-wide burst, overwhelming a service that is recovering but has limited capacity. This can prolong or amplify the original incident.

CONFIDENCE: Very high.

FALSIFIER: The service or an intermediary strictly rate-limits/admission-controls retries below proven recovery capacity.

CLAIM: The policy needs jitter, preferably full jitter, plus retry limits/budgets.

EVIDENCE: Randomizing each delay—such as uniformly between zero and the current exponential cap—decorrelates clients. A five-attempt cap limits individual amplification but does not prevent hundreds of clients from retrying together.

CONFIDENCE: High.

FALSIFIER: Load testing shows deterministic retry peaks remain comfortably below downstream capacity under the maximum credible simultaneous failure.

**VERDICT:** **CALL: No—deterministic exponential backoff is insufficient.** Biggest risk: synchronized retry waves repeatedly overload the recovering downstream. Cheapest discriminating test: simulate all clients receiving a 503 simultaneously and histogram retry requests over time; compare the current policy with full-jitter backoff.
