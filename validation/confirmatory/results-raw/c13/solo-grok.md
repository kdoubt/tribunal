**DECISION MEMO — C13: Kafka vs managed queue**

**CALL**
**B**

**Context.** Order events fan out to a handful of consumers (email, analytics, fulfillment) at ~50/s, burst 500. No streaming or replay requirement today.

**Why B.** That load is a rounding error for SQS/SNS, Cloud Pub/Sub, or a managed broker. The job is reliable async fan-out with retries and DLQs, not a durable partitioned log. Kafka’s value is independent consumer offsets, replay, and ordered partitions at high throughput. None of that is in the requirement. Buying Kafka now pays for brokers, partitions, consumer-group ops, and a platform team for unused features. A managed queue ships in days, not a quarter.

**Tradeoffs.** You give up cheap historical replay and late-joining consumers that catch up from offset 0. At-least-once + idempotent handlers is the contract; exactly-once and strict per-key ordering are weaker than Kafka. If this bus becomes the company event spine, you will migrate once. That is cheaper than running Kafka as a glorified job queue.

**Single biggest risk.** A production bug (or a new analytics consumer) requires reprocessing days of order history, and the queue has already dropped the messages.

**Falsification test (cheap).** For 90 days, log every time a consumer would have needed events older than the queue retention/visibility window (recompute, backfill, late subscriber, “replay last Tuesday”). If that happens **more than once**, or if a new consumer cannot go live without a historical catch-up, you chose wrong — introduce a log (Kafka or equivalent) then, not now. Also abort B if sustained ingest exceeds ~5k/s with per-key ordering SLAs the queue cannot meet.
