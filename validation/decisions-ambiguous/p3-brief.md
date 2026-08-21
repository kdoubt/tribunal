# Decision P3 - pgvector vs a dedicated vector database

A team already running **PostgreSQL** in production is adding a
retrieval-augmented (RAG) feature. Expected corpus: **~10 million embeddings**,
growing; latency target ~tens of ms for top-k; a small team that does not want to
operate more stateful systems than necessary. They must choose the vector store:

- **A) Postgres + pgvector** - keep vectors in the DB they already run and back
  up; one system, transactional with their metadata; HNSW indexing available.
- **B) A dedicated vector database** - a purpose-built ANN store (e.g. a managed
  or self-hosted vector DB) optimized for large-scale similarity search and
  filtering, at the cost of a new stateful system to run and sync.

No consensus at ~10M vectors; the crossover is genuinely arguable. **Commit to A
or B**, name the single biggest risk of your choice, and give the cheapest test
or threshold that would tell this team they chose wrong.
