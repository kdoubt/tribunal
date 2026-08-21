# C01 sealed truth - SEALED (arms never receive this path)

- **difficulty:** harder known-answer (a strong model can miss the failure modes)
- **correct_call:** **NOT sufficient as-is.** `CREATE INDEX CONCURRENTLY` avoids
  the long write-lock (good), but it can **fail** - most commonly on **existing
  duplicate values**, or if interrupted / a conflicting long transaction - and on
  failure it **leaves behind an INVALID index** that is NOT enforcing uniqueness
  and still costs storage/maintenance. The team must: check
  `pg_index.indisvalid` (or `\d`), and if invalid, `DROP INDEX` and retry after
  removing duplicates. Also: it cannot run inside a transaction block, does two
  table scans, and waits on existing long transactions.
- **oracle:** PostgreSQL docs, "Building Indexes Concurrently" (CREATE INDEX) -
  "if a problem arises ... an invalid index is left behind ... it will not be
  used ... recommended to drop it and try again."
- **must_catch:**
  1. CONCURRENTLY can fail (esp. on existing duplicates) and **leave an INVALID
     index** that does not enforce uniqueness;
  2. so you must **verify `indisvalid`** and DROP+retry on failure;
  3. it **cannot run in a transaction block** and waits on long-running txns;
  4. (bonus) a UNIQUE index enforces going forward but does nothing about
     duplicates already present - dedupe first.
- **landmine (confident wrong answers):** "sufficient as-is - CONCURRENTLY makes
  it safe and online, done"; assuming a failed run cleans up after itself; not
  mentioning the invalid-index-left-behind state or the `indisvalid` check.
