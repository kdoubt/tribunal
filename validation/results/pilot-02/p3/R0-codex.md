**CLAIM 1 — Choose A: pgvector is credible at 10M vectors, and scale alone does not justify another datastore.**  
**EVIDENCE:** pgvector provides HNSW ANN indexes with tunable `ef_search`, explicitly trading latency for recall. PostgreSQL partitioning can further constrain searches when tenant or corpus boundaries naturally permit pruning. ([pgvector documentation](https://github.com/pgvector/pgvector), [PostgreSQL partitioning](https://www.postgresql.org/docs/current/ddl-partitioning.html))  
**CONFIDENCE:** 0.83  
**FALSIFIER:** Production-shaped testing cannot simultaneously achieve the required recall and p95 latency on affordable hardware.

**CLAIM 2 — A materially reduces correctness and operational risk.**  
**EVIDENCE:** Vectors and metadata participate in the same PostgreSQL transactions, backups, replication, authorization, and deletion workflow. B requires synchronization or dual writes, creating stale-vector, orphan, recovery-order, and consistency failure modes.  
**CONFIDENCE:** 0.94  
**FALSIFIER:** The team already has a reliable change-data-capture platform and a fully managed vector service whose operational burden is demonstrably negligible.

**CLAIM 3 — Filtering is the likely crossover trigger, not the raw 10M count.**  
**EVIDENCE:** pgvector documents that approximate-index filtering is applied after scanning candidates; selective filters may return too few matches. Iterative scans, scalar indexes, partial indexes, and partitioning mitigate this, but increase tuning complexity and work. ([pgvector filtering guidance](https://github.com/pgvector/pgvector#filtering))  
**CONFIDENCE:** 0.90  
**FALSIFIER:** Representative filtered queries maintain target recall and latency across worst-case tenants and predicates without excessive scan expansion.

**CLAIM 4 — Growth can pressure the existing database before retrieval visibly fails.**  
**EVIDENCE:** HNSW gains speed through a memory-intensive graph; pgvector notes that builds slow substantially when the graph exceeds `maintenance_work_mem`. Large indexes also compete with transactional workloads for RAM, cache, I/O, vacuum, and replicas.  
**CONFIDENCE:** 0.88  
**FALSIFIER:** Load tests show stable OLTP latency, replica lag, maintenance time, and ANN latency at projected 2–3× corpus size.

**VERDICT: A — Postgres + pgvector.** The biggest risk is HNSW memory/I/O contention causing filtered-query tail latency or harming the existing OLTP workload as the corpus grows.

The cheapest discriminating test: load a production-shaped 10M-vector snapshot on equivalent hardware, replay representative concurrency and filters, and compare ANN results with exact-search ground truth. Choose B if either **p95 exceeds the team’s concrete “tens of ms” SLO (use 50 ms if unspecified), recall@k falls below 95%, or existing PostgreSQL p95 latency regresses by more than 10%** at projected 12-month load.
