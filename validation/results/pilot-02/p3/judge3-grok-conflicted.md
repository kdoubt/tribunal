Memo X: r1=3 r2=3 r3=3 r4=3 total=12/12 — Commits to A with the real load-bearing risk (filtered tails + OLTP contention as the graph falls out of RAM) and a pre-registerable fail condition (cannot hit recall + p95 ≤80 ms on affordable cached-HNSW hardware, or memory residency breaks).

Memo Y: r1=3 r2=3 r3=3 r4=2 total=11/12 — Same A call and a sharper dual-SLO falsifier (recall@k + p95 ≤50 ms *and* existing DB p95 within 10%), but a bit more padding on generic dedicated-DB advantages and less tight than X on the memory-residency crossover.

CHOICE: X - X is tighter and names the actual failure mechanism (filtered ANN + graph residency) without watering the call, while Y’s extra SLO coupling is good but slightly padded.
