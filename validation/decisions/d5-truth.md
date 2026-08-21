# D5 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (discriminator candidate)
- **correct_call:** **NO.** `MAX()+1` read then a separate `INSERT` is a
  read-then-write race under READ COMMITTED: two concurrent requests both read
  the same `MAX`, both insert the same `invoice_no` → **duplicates**. (The issue
  is duplicates, not gaps.) With no unique constraint, the duplicates are
  silently persisted.
- **oracle:** PostgreSQL docs on sequences (`nextval` is concurrency-safe and
  atomic) and transaction isolation (READ COMMITTED allows this interleaving).
- **correct fix (any one):**
  1. a **sequence**/`IDENTITY`/`bigserial` for `invoice_no` (atomic `nextval`;
     accept that it may leave gaps on rollback);
  2. a dedicated counter row bumped with `UPDATE counters SET n=n+1 ... RETURNING n`;
  3. at minimum a `UNIQUE` constraint on `invoice_no` **plus** retry - but a
     unique constraint alone only converts the race into insert *failures*, it
     does not make the scheme correct on its own.
- **must_catch:**
  1. `MAX()+1` + separate INSERT has a read-insert race → duplicate numbers;
  2. the missing `UNIQUE` constraint means duplicates persist silently;
  3. a **sequence** / atomic counter is the mechanism (gaps are acceptable for invoice numbers; duplicates are not);
  4. an index/unique constraint alone does not *produce* correct unique numbers - it only turns the race into errors that still need handling.
- **landmine (confident wrong answers):** "yes, MAX+1 is fine"; "just add an
  index/unique constraint and it's correct" (that causes failures, not
  correctness); "wrap in a transaction" (READ COMMITTED still races).
