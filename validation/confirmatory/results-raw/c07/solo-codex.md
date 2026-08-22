## Decision Memo — C07

**CALL:** No. This double-checked-locking singleton is not thread-safe; declare `instance` `volatile` (or use the initialization-on-demand holder idiom).

**Single biggest risk:** **Unsafe publication of a partially initialized `Config`.** A thread may observe `instance != null` and use it without acquiring the monitor, yet still observe stale/default values in its fields. The reference read/write is atomic, but atomicity does not establish visibility or ordering.

**Key risks:**

- The unsynchronized outer read conflicts with the write and has no happens-before relationship: this is a data race.
- The constructor’s field writes are not guaranteed visible to a thread taking the unlocked fast path.
- The failure may appear only under particular JIT optimizations, architectures, or production timing.
- Stress tests passing do **not** prove correctness under the Java Memory Model.

The JLS specifies that monitor release only happens-before a subsequent acquisition of the **same** monitor; the fast path performs no acquisition. Conversely, a write to a `volatile` field happens-before subsequent reads of that field, providing the missing publication guarantee. [JLS §17.4.4–§17.4.5](https://docs.oracle.com/javase/specs/jls/se26/html/jls-17.html#jls-17.4.5), [JLS §8.3.1.4](https://docs.oracle.com/javase/specs/jls/se26/html/jls-8.html#jls-8.3.1.4)

**Cheapest wrong-choice criterion:** Perform a happens-before audit. If any thread can read the published `instance` without either a volatile read or acquiring the publishing monitor, reject the implementation. That criterion already fails here; no runtime test is needed.

```java
private static volatile Config instance;
```

With `volatile`, this double-checked-locking pattern is valid, assuming the constructor does not leak `this`.
