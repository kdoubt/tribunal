DECISION MEMO — C13

CALL

B

Choose a managed queue. At 50 events/sec with bursts to 500, Kafka’s durable log, partitioning model, and replay capabilities solve requirements the team does not currently have while adding operational and application complexity.

Key tradeoffs and risks:

- Managed queues minimize infrastructure work, simplify scaling, and provide adequate durability, retries, dead-lettering, and fan-out for email, analytics, and fulfillment.
- Use separate subscriptions/queues per consumer so failures and backpressure are isolated.
- Delivery will typically be at least once. Consumers must be idempotent, tolerate duplicates, and handle out-of-order events where ordering is not explicitly guaranteed.
- A queue offers weaker historical replay and event-retention semantics. Reprocessing old events may require rebuilding from the system of record.
- Kafka would provide stronger replay, ordering within partitions, and a better foundation for stream processing, but those benefits do not justify its operational and conceptual cost today.

Single biggest risk:

The organization later discovers that reliable historical replay is a core requirement—for audit recovery, analytics backfills, or rebuilding downstream state—and the queue’s retention window or semantics cannot support it.

Cheapest test/criterion that tells the team they chose wrong:

Run a requirements review with every consumer owner and ask: “Must we be able to reconstruct this consumer from all order events after its data is lost or its logic changes?” If any committed use case requires replay beyond the managed service’s practical retention window, or requires independently rewinding consumers without reading the order database, B is wrong and the team should choose Kafka.
