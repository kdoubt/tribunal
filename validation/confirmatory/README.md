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
- `results/` - added after the run, every decision published including losses.

## Status: RUN COMPLETE (n=20) - no lift confirmed

Full results in [`RESULTS.md`](RESULTS.md). The independent judge (`gpt-oss-120b`,
temp 0, neither seat nor orchestrator vendor) is wired and was used to score the
ambiguous arm. Headline: **seats disagreed on only 1 of 20 decisions; the panel
never out-scored a single strong model; Round 1 fired once (c09) and correctly
adjudicated.** The no-lift prior is confirmed at n=20 with an independent judge,
oracle-scored known-answer decisions, and component-isolating arms.

Still open (stated honestly in `RESULTS.md`): the **non-OpenAI second judge** the
protocol calls for was not added (gateway key unavailable), the ambiguous panel
memo was crudely rendered, and only one decision exercised Round 1 - so the
primary endpoint rests on a single data point. A larger, two-judge, real-outcome
study remains the ceiling; this run is a genuine confirmatory attempt, not the
last word.

The harness scripts (arm runner, judge caller) live operator-side, not in this
docs-only repo; only the protocol, decision set, and results are committed here.
