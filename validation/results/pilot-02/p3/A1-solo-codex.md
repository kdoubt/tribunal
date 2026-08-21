# Decision Memo: Choose A — Postgres + pgvector

**Recommendation:** Use PostgreSQL with pgvector.

At ~10 million embeddings, pgvector is within a credible operating range, and the team’s strongest constraint is operational capacity—not theoretical ANN scalability. Reusing PostgreSQL preserves one backup, recovery, security, observability, and on-call model. It also keeps vectors and metadata transactionally consistent, avoiding synchronization pipelines and failure modes between the source of truth and a dedicated vector store.

The tradeoff is reduced performance headroom. A dedicated vector database may deliver better tail latency, filtering, index-build behavior, and horizontal scaling as the corpus and query rate grow. pgvector’s HNSW index also adds substantial memory, storage, vacuum, replication, and rebuild costs; sharing infrastructure with transactional workloads increases contention risk. These costs are acceptable now because operational simplicity is immediate and certain, while the dedicated system’s advantages are workload-dependent.

**Single biggest risk:** pgvector cannot sustain the required filtered-search recall at tens-of-milliseconds tail latency under realistic concurrent load without overprovisioning PostgreSQL or harming existing production workloads.

**Cheapest falsification test:** Before launch, benchmark a production-shaped snapshot—ideally all 10 million vectors, with real dimensions, metadata-filter selectivity, query distribution, and expected concurrency—on the intended PostgreSQL instance. Measure recall against exact search and p95/p99 latency while normal transactional traffic runs.

Treat A as the wrong choice if, after reasonable HNSW and query tuning, it cannot simultaneously achieve the team’s minimum recall and latency SLO—for example, **≥95% recall@k and p95 ≤50 ms at projected peak concurrency—while keeping existing database p95 latency within 10% of baseline**. If it fails that criterion, move to a managed dedicated vector database.
