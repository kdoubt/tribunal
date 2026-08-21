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

## Status: harness proven; set + run pending

The independent judge is wired and **verified working**: `gpt-oss-120b` on a
local OpenAI-compatible endpoint (neither seat, nor the orchestrator's vendor),
temperature 0 for reproducibility. Its first use was a robustness re-judging of
the pilot-02 memos, and it **corroborated the no-lift finding** - a dead tie on
aggregate (32-32), not a panel win (see [`../results/pilot-02.md`](../results/pilot-02.md)).
So the fourth-vendor-judge blocker that made pilot-02 exploratory is now solved.

Remaining before the confirmatory scoring run:

1. **Finish the decision set to n ≥ 20** (harder known-answer decisions where a
   strong solo can err + ambiguous no-ground-truth calls + a real-outcome
   subset), each with sealed truth.
2. **Add the designated non-OpenAI second judge** (Meta Llama or Alibaba Qwen via
   the gateway) once a key is available, per the protocol's two-judge rule.
3. **Run** the four arms per decision, judge blind under both judges, score, and
   publish - lift or no lift.

The harness scripts (arm runner, judge caller) live operator-side, not in this
docs-only repo; only the protocol, decision set, and results are committed here.
The prior on the primary endpoint, from two pilots and now an independent judge,
is **no lift** - this study exists to overturn or confirm that honestly.
