## Decision memo — Choose A: PostgreSQL + pgvector

**Recommendation:** Store the ~10M embeddings in the Postgres you already run,
using pgvector with an HNSW index; do not stand up a dedicated vector database
yet.

**Why A, at this scale and for this team:**

- **~10M vectors is within pgvector's demonstrated operating range.** Aurora
  PostgreSQL + pgvector 0.8.0 on a 10M-item corpus: ~13 ms unfiltered at
  `ef_search=40`, and filtered e-commerce queries in the ~70–160 ms band
  depending on `ef_search` (AWS, 2025). An independent 10M × 1536-d report shows
  8–15 ms top-10 at ~95% recall on an `r7g.4xlarge` (2026). This meets a
  tens-of-ms target if the HNSW graph is memory-resident.
- **One system, not two.** Reusing Postgres preserves a single backup, recovery,
  security, observability, and on-call model, and keeps vectors transactionally
  consistent with their metadata — no source-of-truth-to-index sync pipeline and
  its failure modes. The team's binding constraint is operational capacity, not
  theoretical ANN scalability.
- **The recall/latency knob is explicit.** pgvector's HNSW exposes tunable
  `ef_search` (trade latency for recall); Postgres partitioning can prune
  filtered searches when tenant/corpus boundaries allow.

**Single biggest risk:** HNSW memory/I/O contention causing **filtered-query
tail latency to miss target — or harming the existing OLTP workload** — as the
corpus grows past the point where the graph stays resident in RAM. Filtered ANN
(a `WHERE` predicate plus vector search) is where pgvector is weakest and where
scan expansion bites.

**Cheapest test that says you chose wrong:** a production-shaped load test on the
real corpus with **realistic filter predicates and worst-case tenants**, at
target concurrency, measuring p95 latency and recall together. Choose the
dedicated vector DB (B) only if you **cannot** simultaneously hit target recall
and p95 (~≤80 ms) on affordable hardware with the HNSW graph cached — or if
projected corpus growth blows the memory budget that keeps it resident. If the
test passes, B is the overcommit: a second stateful system to run and sync for
no measured gain.
