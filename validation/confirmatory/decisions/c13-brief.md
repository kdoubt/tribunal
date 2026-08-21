# Decision C13 - Kafka vs a managed queue

A team needs **asynchronous event processing** (order events -> a few consumers: email, analytics, fulfillment) at ~50 events/sec, bursting to 500. No current streaming/replay requirement. Choose:

- **A) Apache Kafka** (durable log, replay, partitioning; ops overhead).
- **B) A managed queue** (SQS / RabbitMQ / cloud pub-sub; simpler, less replay).

Commit to A or B, name the single biggest risk, and give the cheapest test/criterion that would tell them they chose wrong.
