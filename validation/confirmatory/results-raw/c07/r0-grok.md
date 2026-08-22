**Not thread-safe.** Classic broken DCL. `instance` must be `volatile` (or use the holder class).

**CLAIM 1:** This DCL is not thread-safe on any Java Memory Model the JVM actually implements (JSR-133 / Java 5+).
**EVIDENCE:** The unsynchronized outer read of `instance` has no happens-before with the write inside the lock. `synchronized` only orders threads that enter the monitor; a reader that sees a non-null reference never enters it.
**CONFIDENCE:** 0.95
**FALSIFIER:** A JLS/JMM cite that an unsynchronized read of a non-volatile reference published under a lock still happens-before the constructor writes.

**CLAIM 2:** The assignment `instance = new Config()` can publish a reference before constructor stores are visible.
**EVIDENCE:** Allocation, field init, and store of the reference are not a single atomic action. Compiler/JIT may reorder stores; the CPU may too on weak arches. A racy reader can pass `instance == null` and use a half-built `Config`.
**CONFIDENCE:** 0.93
**FALSIFIER:** Show that `new` + store to a non-volatile field is a freeze (no store-store reordering out of the constructor) on all JVM targets.

**CLAIM 3:** `volatile` (or `AtomicReference` / class-init holder) is the fix, not “it works on x86.”
**EVIDENCE:** `volatile` write of the reference happens-before later volatile reads and forbids reordering constructor stores past publication. Initialization-on-demand holder is safe because class init is internally locked. x86 TSO does not bind the JIT.
**CONFIDENCE:** 0.9
**FALSIFIER:** A spec/JVM guarantee that TSO plus HotSpot never reorders constructor stores past a non-volatile publish.

**CLAIM 4:** “Stale forever-null” is real but secondary; the production killer is a live, wrong object.
**EVIDENCE:** A thread can cache `instance == null` and retry, or (worse) cache a non-null ref to uninitialized fields. Callers then see default zeros/nulls, torn config, or NPEs that vanish under a debugger.
**CONFIDENCE:** 0.85
**FALSIFIER:** Telemetry showing the only observed failure mode is livelock on the outer null check, never a published-but-uninit instance.

**VERDICT:** CALL **unsafe — do not ship.** Biggest risk: **racy reader gets a published, incompletely initialized `Config`.** Cheapest test: **JCStress DCL fixture** (or `-XX:+PrintAssembly` + ARM/weak-memory run) asserting all `Config` fields match constructor stores after a non-null `get()`; it will fail without `volatile`.
