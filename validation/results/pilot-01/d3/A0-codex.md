**CALL: Not safe.**

**Reasoning:** Webhook delivery is typically *at least once*. The ledger insert can commit, but the HTTP 200 can be lost due to a timeout, dropped connection, proxy failure, or process crash. The provider then retries the same successful charge, causing another insert and credit.

**Single biggest risk:** A committed credit followed by a lost/unobserved acknowledgment, resulting in duplicate delivery and double-crediting.

**Fix:** Make processing idempotent: store the provider’s immutable charge/event ID in the ledger with a `UNIQUE` constraint, and use an atomic insert/upsert transaction so retries become no-ops. Return 2xx after that transaction commits.
