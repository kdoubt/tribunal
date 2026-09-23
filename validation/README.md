# validation/ - does the panel actually beat a single model?

Three independent reviews of Tribunal agreed on one thing: the repo shows the
process *runs*, but not that it produces **lift** over a single strong model
plus tools. This directory is the honest attempt to measure that - or to find
that there is no lift and say so.

- [`PROTOCOL.md`](PROTOCOL.md) - the **pre-registered** ablation: four arms (solo
  / solo+oracle / two isolated seats / full Tribunal), fixed scoring, and the
  comparisons that matter (does *debate* beat one-model-plus-oracle?).
- [`decisions/`](decisions/) - the pre-registered decision set. Each `dNN-brief.md`
  is what every arm sees; each `dNN-truth.md` is the **sealed** rubric (correct
  call + must-catch + landmine, backed by a named oracle) used only for scoring.
  The briefs are known-answer engineering decisions with a difficulty spread:
  controls that all arms should get right, plus subtle concurrency/idempotency
  traps where a confident single model may slip.
- `results/` - the pilot runs, with raw arm outputs and scores. Every decision
  is reported, including panel losses and false objections.
- [`confirmatory/`](confirmatory/) - the **pre-registered study labeled
  "confirmatory"**, designed to fix the four reasons the pilots were only
  exploratory (a judge independent of the seats and orchestrator (a fourth
  model, OpenAI-lineage), component-isolating arms, mechanical panel output,
  n ≥ 20 with a disclosed scorer and access-separated blinding). **Run complete
  (n=20): no lift observed** - see
  [`confirmatory/RESULTS.md`](confirmatory/RESULTS.md), including its opening
  caveat on how the execution fell short of "confirmatory".

- [`FIELD-RECORD.md`](FIELD-RECORD.md) - what one month of the maintainer's
  own panel archive shows (28 runs, exported with `flywheel-export`).
  Observational and self-reported, not lift evidence; it quotes that Round 1
  ran in 7 of 28 exported runs against 1 of 20 in the study, without treating
  those rates as the same measurement.

- [`confirmatory/SECOND-JUDGE.md`](confirmatory/SECOND-JUDGE.md) - the **designated
  non-OpenAI second judge**, run 2026-09-22, the one pre-registered step that had never
  been executed. Forced-choice agreement with the primary judge: 9 of 10, both panel 0.
  It does **not** complete the primary endpoint and does **not** declare or refute lift.
  Raw per-decision scores and replies are published beside it.

- [`adapter-probes.md`](adapter-probes.md) - the **re-runnable probes behind the
  adapter rules**: dated vendor-CLI transcripts (a seat whose read path is shell
  refusing a "don't use shell" read-test; a silent exit-0 outside a trusted
  directory). Operational evidence for `adapters/`, not lift evidence and not a
  benchmark; carries no panel content, so the transcripts are verbatim.

- [`grounded/`](grounded/) - the **pre-registered grounded ablation**: ten
  decisions from the maintainer's archive with known outcomes (nine on real
  artifacts, one brief-only control), three vendors after Amendment 1
  withdrew the local Qwen seat (registered with four) as solo arms and
  rotating judges, a two-seat panel arm, and new `grounding_rate` and
  `support_rate` metrics. Protocol
  and sealed rubrics were committed before any arm ran; the artifacts are
  private, so what is replicable is the protocol and the published scored
  outputs.
  **Run complete (three vendors, ten decisions): no lift** - see
  [`grounded/RESULTS.md`](grounded/RESULTS.md): paired panel-minus-union
  must-catch −0.18 (n=10), decision correctness −0.06 strict / +0.06
  sensitivity (n=9), false objections 1.10 vs the union's upper-bound 2.60;
  Round 1 ran on all ten, overturned no claim, and changed no verdict input's
  direction.

This is a pilot harness, stated as such. A handful of decisions is not proof;
it is the first real evidence, built so it can scale to the ~20+ real-world runs
the reviews (correctly) set as the bar for calling Tribunal *validated* rather
than *promising*.

> Why the first decision sets were public/known-answer: a published ablation
> needs ground truth anyone can check and outputs anyone can re-score. The
> grounded follow-up now reports nine artifact-backed private decisions and one
> brief-only control with recorded outcomes; its artifacts stay private, its
> scored outputs are published.
