**CALL: not safe.**

The provider guarantees **at-least-once**, not exactly-once, delivery. The ledger insert can commit, but the connection can fail before the provider receives the 200; it then retries the same successful charge. HTTP explicitly recognizes this ambiguity: a request may have succeeded even though its response was not received ([RFC 9110 §9.2.2](https://www.rfc-editor.org/rfc/rfc9110.html#section-9.2.2)).

**Single biggest risk:** a lost 200 after the ledger commit causes the retried webhook to insert a second credit.

**Fix:** make processing idempotent. Store the provider’s immutable charge/event ID on the ledger row with a database `UNIQUE` constraint, and atomically insert/credit in one transaction. On conflict, treat the delivery as already processed and return 2xx. The database constraint—not a prior “does this exist?” query—must enforce deduplication under concurrent deliveries.
