# confirmatory/ - the study built to actually settle the lift question

The two [pilots](../results/) were exploratory and showed no lift, but a
re-review correctly noted they could not *confirm* anything: the judge shared the
orchestrator's vendor, the arms didn't isolate components, the panel memo was
orchestrator-assembled, and n was tiny with an undisclosed scorer and
instruction-only blinding. This directory is the pre-registered study that
removes all four, so its result is load-bearing either way.

- [`PROTOCOL.md`](PROTOCOL.md) - the full pre-registration: component-isolating
  arms (oracle held constant, so `A_panel − A_2seat` isolates Round 1), an
  **independent fourth-vendor judge**, an independent scorer, access-separated
  blinding, n ≥ 20, both metrics with **aggregate score as the primary
  endpoint**, and a pre-registered decision rule for declaring lift.
- `decisions/` + `sealed/` - the fixed decision set (briefs the arms see; sealed
  oracle/truth the seats never receive). Committed before scoring.
- [`RESULTS.md`](RESULTS.md) - the writeup, every decision including losses.
- [`results-raw/`](results-raw/) - raw arm transcripts (the decisive c09 chain
  in full) and the per-decision judge log, so the scores are auditable.

## Status: RUN COMPLETE (n=20) - no lift observed

Full results in [`RESULTS.md`](RESULTS.md). The independent judge (`gpt-oss-120b`,
temp 0, independent of the seats and the orchestrator vendor) scored the ambiguous
arm. Headline: **seats disagreed on only 1 of 20 decisions; the panel never
out-scored a single strong model; Round 1 fired once (c09) and correctly
adjudicated.** No lift was observed at n=20 under a judge independent of the seats
and orchestrator, with oracle-scored known-answer decisions and
component-isolating arms.

**Deliberately out of scope for this run** (left to replicators, not gaps to close
before shipping): a **non-OpenAI second judge** (the gateway key was not
available), a polished rather than mechanical ambiguous-panel rendering, and
powering the Round-1 sub-endpoint beyond its single data point. A larger,
two-judge, real-outcome study is the natural *replication*, not something this
frozen repo will do itself.

The harness scripts (arm runner, judge caller) live operator-side, not in this
docs-only repo; only the protocol, decision set, and results are committed here.
