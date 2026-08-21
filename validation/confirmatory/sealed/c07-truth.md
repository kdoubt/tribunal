# C07 SEALED
- correct_call: **NOT thread-safe.** Without `volatile`, double-checked locking is
  broken under the Java Memory Model: another thread can observe a **non-null but
  partially-constructed** `instance` (reordering of the write to `instance` before
  the constructor's writes are visible).
- oracle: JLS / "The 'Double-Checked Locking is Broken' Declaration"; JSR-133.
- fix: declare `instance` `volatile` (works since Java 5), or use a holder-class
  idiom / enum singleton.
- must_catch: [non-volatile DCL is broken; partially-constructed object visibility / reordering; volatile fixes it since Java 5; holder-idiom alternative]
- landmine: "thread-safe, the synchronized + null checks cover it" / missing the volatile requirement.
