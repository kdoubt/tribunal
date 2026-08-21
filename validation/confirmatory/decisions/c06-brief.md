# Decision C06 - retry policy for a shared downstream

Hundreds of clients call a shared service. On 503s, each client retries with
**exponential backoff** (1s, 2s, 4s, ...), capped at 5 tries. During an incident
the downstream briefly fails for everyone at once. **Question:** is exponential
backoff (as described) sufficient to avoid making the incident worse? Commit and
name the single biggest risk.
