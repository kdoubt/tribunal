# Confirmatory study - results (n=20)

> **Read this as a strong *pre-registered follow-up*, not a clean confirmatory
> ablation.** It has real confirmatory elements (pre-registration, isolating arms,
> a judge independent of the seats and orchestrator, oracle-scored known-answer
> decisions), but also real deviations that keep it short of "confirmatory": an
> **ex-post oracle-picked best-of-two solo baseline**, a **single** judge (a
> fourth *model*, gpt-oss, but OpenAI-lineage - not a fourth *vendor*), **no live
> oracle actually invoked**, an ambiguous panel memo rendered **cruder than the
> pre-registered template fill** (with the rendering control not run), and the
> pre-registered Round-1 endpoint resting on **n=1**. See Limitations. The
> "confirmatory" name is the study's label, not a claim that it settled the
> question. (This caveat was added after an external re-review pushed back on the
> word.)

**Run:** 2026-08-21. **Seats:** Codex CLI (GPT-5-class), Grok CLI (Grok-4-class).
**Independent judge:** gpt-oss-120b (a fourth *model*, independent of the seats
and orchestrator; OpenAI-lineage, temp 0). **Decisions:** the pre-registered set
(10 known-answer with sealed oracle truth; 10 ambiguous). **Design:**
[`PROTOCOL.md`](PROTOCOL.md), committed before the run. Raw arm transcripts for
all 20 decisions are in [`results-raw/`](results-raw/); the blind X/Y-to-arm
mapping, the verbatim judge prompt, and a score reconciliation are in
[`results-raw/AUDIT.md`](results-raw/AUDIT.md).

## Headline

**No lift observed (single independent judge) - and the panel's engine almost never activates.**

- **Seats disagreed on 1 of 20 decisions.** On the other 19 the two vendors
  independently reached the same call, so Round 1 never ran and `A_panel = A_2seat`.
  This is the study's most robust, judge-independent finding, and it confirms the
  pilots at n=20: **these two frontier models mostly converge, so the panel
  machinery idles.** (A rate for these two seats on this decision set - not an
  established rate for frontier models in general.)
- **On the 1 disagreement (c09), Round 1 resolved the split** to the
  sealed-correct call. Read the fine print below before crediting it as
  "correcting a wrong seat": the split was over the *scope* of the question
  (portable vs PostgreSQL-specific), and the seat scored wrong had stated the
  correct mechanism and recommended the sealed truth's own fix. A single
  data point for the mechanism (n=1), with a contestable classification -
  not a demonstration.
- **The panel never out-scored a single strong model.** On known-answer it *tied*
  the best-of-two solo (10/10 each); on ambiguous the independent judge preferred
  the solo memo (aggregate 118 vs 103; 9 of 10 forced choices).

## Known-answer arm (n=10, oracle-scored)

`decision_correct` against the sealed `correct_call`:

| arm | score | notes |
|----|----|----|
| Codex solo | **9 / 10** | scored wrong on c09: a PostgreSQL-qualified YES headline against the sealed "not (by itself) safe" - a contestable classification (see below) |
| Grok solo | **10 / 10** | correct on c09 |
| A_solo (better of the two, per decision) | **10 / 10** | grok carries c09 |
| A_2seat (two seats, no debate) | 9/10 agreed+correct; **c09 = unresolved split** | no consensus on c09 |
| **A_panel (Round 1 where disputed)** | **10 / 10** | c09 resolved correctly by Round 1 |

Seats agreed on 9/10; **c09 was the only disagreement - and it was a
scope-framing split, not a clean right-vs-wrong.** The brief did not name a
database. Grok answered "**NO**" on the portable question. Codex (both solo and
seat) answered "**YES - on PostgreSQL**", and its own memo stated the same
decisive mechanism the sealed truth names (the losing transaction gets a `40001`
serialization abort; the *whole* transaction must be retried; PostgreSQL does
not retry automatically) *and* recommended the sealed truth's own fix (the
atomic conditional `UPDATE ... WHERE stock > 0 RETURNING`, explicitly advising
against shipping the isolation-only change). Scored against the sealed
`correct_call` ("NOT (by itself) safe / not the right fix"), the portable NO is
correct and the PostgreSQL-qualified YES is wrong - but that classification is
contestable: on PostgreSQL specifically, Codex's claim that both conflicting
decrements cannot commit is supported by the PostgreSQL documentation. In
**Round 1, Codex revised YES → NO for the brief as worded** ("the answer to the
brief... is NO"); Grok held; the panel converged on the portable call. So Round
1 resolved the split and produced the sealed-correct recommendation - read it
as a scope clarification of a defensibly-qualified answer at least as much as
the correction of a wrong seat. (An earlier version of this file said Codex
"wrongly said REPEATABLE READ makes the oversell safe" and credited the `40001`
mechanism to Grok alone; the raw transcripts do not support that framing -
corrected. Full transcript: [`results-raw/c09/`](results-raw/c09/).)

**What c09 does and does not show:**

- **Round 1 resolved a split the no-debate arm left open** (`A_panel − A_2seat`
  on c09: a committed sealed-correct call vs an unresolved split). The debate is
  what turned the split into a single portable answer.
- **The panel did NOT beat the best single model.** Grok-solo alone was already
  scored correct on c09. The panel only beat the *weaker fixed vendor* (Codex)
  as scored. Since you cannot know in advance which vendor is right on a given
  decision, the value the run suggests is a **hedge against vendor choice** -
  the panel matched Grok's call and moved Codex's - not an accuracy gain over
  best-of-breed.
- **The hedge's single data point is itself soft.** "Caught the wrong vendor"
  rests entirely on classifying Codex's PostgreSQL-qualified answer as wrong,
  which is disputable (above). At n=1, on a contestable classification, the
  decidable-call hedge is a supported design rationale, not an established
  property.

## Ambiguous arm (n=10, independent judge)

On all 10 decisions the two seats **agreed** (no Round 1). Judged solo memo vs a mechanically-assembled
panel memo, blind, by gpt-oss (both metrics):

- **Aggregate rubric score: solo 118, panel 103 - solo wins.**
- **Forced choice: solo 9, tie 1, panel 0.**

The designated non-OpenAI second judge was run over this same arm on 2026-09-22 and
agreed with the forced choice on **9 of 10** decisions (both judges: panel 0). It does
not complete the primary endpoint and does not declare or refute lift - see
[`SECOND-JUDGE.md`](SECOND-JUDGE.md).

The blind mapping, verbatim judge prompt, and per-decision reconciliation are
published in [`results-raw/AUDIT.md`](results-raw/AUDIT.md).

**Three executed deviations on this arm (all disclosed):**

1. **The panel memo was cruder than the pre-registered rendering.** The protocol
   specified a mechanical fill of `core/templates/verdict.md` from the ledger
   (agreement buckets, seat-verbatim claims, the discriminating test, the
   record). What was actually judged was a recommendation line plus each seat's
   final verdict sentence (see any `results-raw/*/panel.md`). An earlier version
   of this file called that concatenation "per the protocol" - it was not; it
   was a deviation *from* the protocol's specified panel output. The 118-103
   comparison is therefore "complete solo memo vs two abbreviated final
   positions", not "solo vs Tribunal's specified final product" - and the
   judge's risk-articulation criterion is exactly where an abbreviated memo
   bleeds points.
2. **The solo memo was Codex-solo, fixed, for all 10 decisions** - not the
   protocol's per-decision higher-scored solo. Note the direction: a fixed
   single vendor is a *weaker* baseline than the pre-registered ex-post
   best-of-two, so this cannot manufacture the no-lift result - but "solo" here
   means "Codex solo", and Grok's solo memos were never judged.
3. **X/Y order was alternated per decision (counterbalanced 5/5), recorded
   before judging** - not randomized as pre-registered.

Because of (1), read this arm as evidence that the panel's *as-executed* output
did not beat a solo memo under this judge - not as a clean measurement of the
specified method. The pilot-02 mirror (an orchestrator-*polished* panel memo,
which may have flattered the panel, still lost or tied on aggregate) is what
keeps the no-lift direction credible; the exact 118-103 margin is
rendering-sensitive and should not be over-read.

## Primary endpoint & decision rule

- **Pre-registered primary endpoint: mean `A_panel − A_2seat` (the value of Round
  1) on aggregate score.** `PROTOCOL.md` names this, and its decision rule was
  "declare lift only if this is positive and holds under both judges." It was
  **not measured as pre-registered**: no numeric paired aggregate was computed
  across decisions. Seats disagreed on only 1 of 20 decisions, so Round 1 ran
  once; on the other 19 the difference is zero by construction (early stop), and
  on c09 the two-seat arm ended in an unresolved split, to which the
  pre-registration assigns no numeric value - so the mean has no computable,
  pre-registered form here. On the qualitative reading the one measurable case
  favored the panel (a committed sealed-correct call vs an unresolved split).
  Either way the pre-registered lift criterion is **not met** - not refuted, but
  unmeasured-as-specified and resting on a single data point, which is a real
  shortfall of this run as a *confirmatory* test (see Limitations). (An earlier
  version of this file wrongly labeled `A_panel − A_solo` as the pre-registered
  primary; corrected.)
- **Headline practical comparison: `A_panel − A_solo` (does the whole panel beat
  a single model).** It is **≤ 0** - a tie on known-answer against an
  *ex-post, oracle-picked* best-of-two solo (see the baseline caveat in
  Limitations), and negative on ambiguous under the independent judge. So the
  panel does not beat a strong single model. This is the number most readers mean
  by "lift," and it shows none.
- The no-lift prior from the pilots **holds, now under a confirmatory attempt at
  n=20 with a judge independent of the seats and orchestrator, oracle scoring, and
  component-isolating arms** - though under a *single* judge vendor (the
  pre-registered non-OpenAI second judge was not added; see Limitations), so read
  this as "no lift observed in a stronger confirmatory attempt," not "proven for
  all time."

## The operational tax (a first-class finding)

**Grok headless narration-death was pervasive.** With a prompt that merely
*mentioned* consulting docs, Grok tried to use tools and returned narration-only
(no answer) on **10 of 10** known-answer decisions and several ambiguous ones; it
also tried to "load the boardroom skill" on one. It only became reliable under a
strict "answer from your own knowledge; do NOT use tools, load skills, or read
files" prompt - and even then wobbled. **The two-vendor panel carries a real,
large reliability cost in headless/automated use**; a naive operator would get
silent dead seats and no valid panel. This corroborates and sharpens the external
reviewers' "operational friction" critique with hard numbers.

## Limitations (honest) - and why this is a *pre-registered follow-up*, not a clean confirmatory ablation

Despite the pre-registration and the isolating arm design, several deviations
mean this run should be read as a **strong, pre-registered follow-up** rather than
a completed confirmatory ablation (an external re-review, correctly, pushed back
on the word "confirmatory"):

- **The solo baseline is ex-post.** `A_solo` = the *stronger* of the two solo runs
  *picked per decision*. On known-answer that means picked by the oracle (i.e.
  after knowing the right answer), which is **not achievable prospectively** - in
  real use you cannot know which vendor to trust in advance. So "the panel matched
  the best single model" is against an oracle-picked best-of-two, an unfairly
  strong baseline; against a *fixed* single vendor the panel was ≥ that vendor,
  which is the honest, weaker claim.
- **The rubric ceilings on the solo arm, so the score gap is panel-driven.** Solo
  scored the 12/12 maximum on **9 of 10** decisions for the primary judge (solo mean
  11.8), and on **10 of 10** for the designated second judge (solo mean 12.0). The
  comparison arm therefore has almost no room to move: essentially all of the
  aggregate difference - both the primary judge's 118-vs-103 and the second judge's
  wider gap - comes from how harshly each judge marks the *panel* arm, not from any
  movement in the solo arm. Read the **forced choice** as the informative quantity and
  treat the score gap as a weak magnitude estimate. This was **not** pre-registered, and
  - stated plainly - **it was not hidden either**: the primary judge's own decoded
  scores in [`results-raw/AUDIT.md`](results-raw/AUDIT.md) have published
  `12,12,12,12,10,12,12,12,12,12` since this study ran. The ceiling was visible in
  already-published data and nobody, the author included, looked at the distribution
  until the second judge's run prompted it. The second judge widened the pattern
  (10 of 10) but did not reveal it. Recorded here post hoc and cross-referenced from
  `PROTOCOL.md`. It does not change the reported arithmetic.
  *(Corrected in v1.1.13: v1.1.10 asserted this "was only visible once a second judge
  existed", which the AUDIT table contradicts. The error was caught by a seat reviewing
  a draft write-up of this finding.)*
- **The pre-registered primary endpoint (`A_panel − A_2seat`) rests on n=1.** Round
  1 fired on a single decision, so the pre-registered lift criterion is untestable,
  not cleanly satisfied or refuted.
- **No live oracle was actually invoked.** The protocol says oracle access is
  "held constant" across arms; in practice no arm ran a test/tool - the seats
  reasoned from model knowledge (Grok cannot use tools headlessly, see below).
  Oracle access was uniform but *unused*, a deviation from the intended design.
- **n = 20; one judge vendor.** The judge is independent of the seats (Codex,
  Grok) and the orchestrator (Claude), but is gpt-oss (OpenAI lineage), so not a
  fourth *vendor*, only a fourth *model* not fully vendor-disjoint from the Codex
  seat (Codex is in both arms, so any GPT-style affinity largely cancels). The
  protocol's **non-OpenAI second judge (Meta/Alibaba) was not run in this study** - it
  was run later, on 2026-09-22, and is reported separately in
  [`SECOND-JUDGE.md`](SECOND-JUDGE.md); it agreed with this judge's forced choice on
  9 of 10 and did not complete the primary endpoint. At the time of this write-up the
  two-judge confirmation was a replication
  target, and this result stands under a single independent judge.
- **The ambiguous arm deviated from its pre-registered execution in three ways**
  (panel memo cruder than the specified verdict-template fill; fixed Codex-solo
  instead of per-decision best-solo; alternated rather than randomized X/Y
  order) - see the Ambiguous arm section and
  [`results-raw/AUDIT.md`](results-raw/AUDIT.md). The raw-vs-assembled
  panel-rendering control was also not run, and `must_catch_rate` was not
  tabulated per arm. Raw arm transcripts for all 20 decisions are published
  under [`results-raw/`](results-raw/) so anyone can re-score.
- **Known-answer `decision_correct` was applied by the operator**, comparing
  headline calls to the sealed `correct_call` (the protocol assigned rubric
  application to the judge model + mechanical oracle). Little scorer latitude on
  one-word calls - except c09, whose classification is contestable (above).

## Bottom line

At n=20, with a judge independent of the seats and orchestrator (single vendor),
sealed-rubric-scored known-answer decisions, and arms designed to isolate
components (executed with the disclosed deviations above): **no accuracy lift
over a single strong model was observed in this run.** The panel's engine
(disagreement → Round 1) activated on 1 of 20 decisions; when it did, it
resolved the split to the sealed-correct call (n=1, and a scope-framing split
whose "wrong seat" classification is contestable). The defensible value the
data supports is narrow: **on decidable calls, a hedge that matched or beat
each fixed single vendor in this run - resting on that single soft data point
where they diverged; on ambiguous calls, no better (judged worse) than the solo
memo** - plus a preserved counter-case and a discriminating test on contested
calls, paid for with a substantial operational cost. That is what the README
now claims, and no more.
