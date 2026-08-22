# Decision C07 — Lazy singleton (DCL, non-volatile)

**CALL:** Not thread-safe on the JVM. Do not ship. Use `volatile`, the initialization-on-demand holder, or an enum singleton.

## Why

`instance = new Config()` is not a single publication. The JMM allows:

1. allocate
2. store the reference into `instance`
3. run the constructor (or store field writes) later

The outer unsynchronized `if (instance == null)` can observe a non-null reference with **no happens-before** to the constructor writes. `synchronized` on the writing thread does not help readers who never enter the lock. Without `volatile` (Java 5+), there is no store-store / load-load pairing that freezes construction before publication.

This has been broken since the original DCL pattern. Java 5 `volatile` semantics are what made DCL legal. A plain static field is not that.

## Key risks

- Another thread uses a **partially constructed** `Config` (null/zero fields, broken invariants).
- Another thread caches a stale view of fields forever (no subsequent barrier).
- Heisenbugs: passes locally, fails under load, JIT, or a weaker CPU memory model (ARM, especially).
- “It works in prod for months” is not evidence; the race is rare and architecture-dependent.

## Single biggest risk

**Unsafe publication of a half-initialized singleton.** Callers see `get() != null` and then hit NPEs, wrong config, or corrupt downstream state — including values that never self-heal.

## Cheapest wrong-choice test

JCStress (or a two-thread harness on ARM): Thread A calls `get()`; Thread B spins until `instance != null` then reads a field written last in the constructor (e.g. `ready = true`). **Fail if B ever sees `ready == false`.** If that observation is possible, the implementation is illegal. If you will not run that test, treat non-`volatile` DCL as already failed.
