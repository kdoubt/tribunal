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
- [`confirmatory/`](confirmatory/) - the **pre-registered confirmatory study**
  that fixes the four reasons the pilots were only exploratory (independent
  judge independent of the seats and orchestrator (a fourth model, OpenAI-lineage), component-isolating arms, mechanical panel output, n ≥ 20
  with a disclosed scorer and access-separated blinding). Harness proven (the
  independent judge is wired and already corroborated the pilots' no-lift
  finding); decision set + run pending.

This is a pilot harness, stated as such. A handful of decisions is not proof;
it is the first real evidence, built so it can scale to the ~20+ real-world runs
the reviews (correctly) set as the bar for calling Tribunal *validated* rather
than *promising*.

> Why the decision set is public/known-answer rather than real estate decisions:
> a published ablation needs ground truth anyone can check and outputs anyone can
> re-score. Real private decisions with fuzzy outcomes come later, for realism,
> once the harness shows signal here.
