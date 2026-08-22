**CLAIM 1:** Throughput does not justify a log.
**EVIDENCE:** 50 evt/s, 500 burst, is well inside SQS/Pub-Sub/RabbitMQ. Kafka’s partitions, consumer groups, and disk log pay off at 10k+ evt/s or many independent replay consumers—not three fire-and-forget processors.
**CONFIDENCE:** 90%
**FALSIFIER:** Sustained load or fan-out that saturates a managed queue (backlog growth at 500/s, or FIFO/ordering limits with required per-order sequencing).

**CLAIM 2:** Fan-out here is a queue pattern, not a streaming pattern.
**EVIDENCE:** Email, analytics, fulfillment are independent side-effects of an order. SNS→SQS (or topic exchange / Pub-Sub) gives at-least-once delivery per consumer without broker topology, rebalances, or a schema registry.
**CONFIDENCE:** 85%
**FALSIFIER:** A consumer that must share offsets, compact state, or join streams with other topics.

**CLAIM 3:** Replay is Kafka’s real differentiator, and it is not a requirement.
**EVIDENCE:** Orders already have a source of truth in the DB. New or recovered consumers can backfill from records, not from a log. Building Kafka “for later replay” is inventory of unused capability.
**CONFIDENCE:** 80%
**FALSIFIER:** A consumer that cannot be rebuilt from the DB because the *event* (not the current row) is the contract—audit, billing reconstruction, or a new consumer that must replay all history.

**CLAIM 4:** Kafka’s standing tax exceeds its benefit at this scale.
**EVIDENCE:** Brokers (or MSK/Confluent), partitions, lag, rebalances, disk, and on-call are a permanent cost. The product work is three small workers. Queue ops are nearly zero.
**CONFIDENCE:** 85%
**FALSIFIER:** Team already runs Kafka in-house with spare capacity and a working platform team, so incremental cost is near zero.

**VERDICT: B** Biggest risk: a later consumer needs durable historical replay (events, not current DB rows) and the queue has already dropped messages. Cheapest test: specify one dummy consumer that must rebuild last 30 days of *events* with the DB unavailable—if that is a real requirement, B is wrong; if backfill-from-orders is acceptable, B is right.
