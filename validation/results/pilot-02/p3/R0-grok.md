I'll ground this in current pgvector vs dedicated-store evidence at ~10M vectors, then commit.**VERDICT: A — Postgres + pgvector**

**CLAIM 1.** At ~10M embeddings, pgvector HNSW already hits the “tens of ms” top-k envelope if the index stays in RAM.  
**EVIDENCE.** Aurora PostgreSQL + pgvector 0.8.0 on a 10M-product catalog: unfiltered ~13 ms (`ef_search=40`); filtered ecommerce queries in the 70–160 ms band depending on `ef_search` (AWS, May 2025). Independent 10M × 1536-d report: 8–15 ms top-10 at ~95% recall on `r7g.4xlarge` (JusDB 2026).  
**CONFIDENCE.** 0.78  
**FALSIFIER.** Production-like p95 > ~80 ms at target recall with the HNSW graph fully cached.

**CLAIM 2.** For a small team already on Postgres, the dedicated store’s tax is dual-write consistency and a second stateful runbook, not raw ANN speed — and that tax dominates at 10M.  
**EVIDENCE.** RAG retrieval is almost never naked kNN: ACL, tenant, recency, and citation metadata already live in Postgres. A second ANN store forces ETL/CDC and join-in-app. GCP operator report: production pgvector HNSW is common in the 0–10M band; the live tradeoff is “ETL + eventual consistency vs one backup” (HN, 2025).  
**CONFIDENCE.** 0.82  
**FALSIFIER.** They already have a managed vector service and a working CDC join, so the ops/consistency cost is ~zero.

**CLAIM 3.** The real 10M failure mode is **filtered** HNSW + RAM, not unfiltered kNN.  
**EVIDENCE.** HNSW is graph-in-memory: ~20–25 KB/vector all-in is a common real-world footprint (ClickHouse 2026) → 200GB+ class working set at 10M if unquantized. Pinecone (2024): pgvector HNSW does not pre-filter inside the graph; selective metadata filters historically collapsed recall. pgvector 0.8 `iterative_scan` mitigates; it does not make Postgres a distributed filtered-ANN engine.  
**CONFIDENCE.** 0.80  
**FALSIFIER.** Filters are cheap btree prefilters that leave a large candidate set, or they ship `halfvec`/binary quant and the graph still fits RAM.

**CLAIM 4.** Dedicated ANN starts winning on **write/rebuild** and **≥50M**, not on a 10M read-mostly RAG corpus.  
**EVIDENCE.** Timescale pgvectorscale: 471 QPS at 99% recall on 50M × 1536-d (May 2025). JusDB: purpose-built stores pull ahead past ~50M. Firecrawl/HN: HNSW insert/rebuild pain shows up when vectors churn constantly, not when they are mostly append + query.  
**CONFIDENCE.** 0.75  
**FALSIFIER.** Weekly full re-embed, high delete/insert QPS, or a 12-month plan that is 100M+.

**Biggest risk of A.** The HNSW working set (and `VACUUM`/rebuild) lands on the same Postgres that serves OLTP, and selective filters + RAM pressure quietly miss the latency/recall SLO.

**Cheapest discriminating test.** Load 10M production-dim vectors plus the real filter distribution onto a staging replica with production RAM. Run target RAG QPS **concurrent with** captured OLTP. **Kill A** if (p95 top-k > 80 ms at ≥90% recall) **or** (primary OLTP p95 degrades >20%).
