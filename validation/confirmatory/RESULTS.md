# Confirmatory study - results (n=20)

> **Read this as a strong *pre-registered follow-up*, not a clean confirmatory
> ablation.** It has real confirmatory elements (pre-registration, isolating arms,
> a judge independent of the seats and orchestrator, oracle-scored known-answer
> decisions), but also real deviations that keep it short of "confirmatory": an
> **ex-post oracle-picked best-of-two solo baseline**, a **single** judge (a
> fourth *model*, gpt-oss, but OpenAI-lineage - not a fourth *vendor*), **no live
> oracle actually invoked**, the raw-vs-assembled rendering control not run, and
> the pre-registered Round-1 endpoint resting on **n=1**. See Limitations. The
> "confirmatory" name is the study's label, not a claim that it settled the
> question. (This caveat was added after an external re-review pushed back on the
> word.)

**Run:** 2026-08-21. **Seats:** Codex CLI (GPT-5-class), Grok CLI (Grok-4-class).
**Independent judge:** gpt-oss-120b (a fourth *model*, independent of the seats
and orchestrator; OpenAI-lineage, temp 0). **Decisions:** the pre-registered set
(10 known-answer with sealed oracle truth; 10 ambiguous). **Design:**
[`PROTOCOL.md`](PROTOCOL.md), committed before the run. Raw arm transcripts for
all 20 decisions are in [`results-raw/`](results-raw/).

## Headline

**No lift observed (single independent judge) - and the panel's engine almost never activates.**

- **Seats disagreed on 1 of 20 decisions.** On the other 19 the two vendors
  independently reached the same call, so Round 1 never ran and `A_panel = A_2seat`.
  This is the study's most robust, judge-independent finding, and it confirms the
  pilots at n=20: **frontier models mostly converge, so the panel machinery idles.**
- **On the 1 disagreement (c09), Round 1 worked** - it correctly adjudicated the
  split, moving the *wrong* seat to the right answer. That is a single encouraging
  data point for the mechanism (n=1), not a demonstration; it just rarely gets to
  act at all.
- **The panel never out-scored a single strong model.** On known-answer it *tied*
  the best-of-two solo (10/10 each); on ambiguous the independent judge preferred
  the solo memo (aggregate 118 vs 103; 9 of 10 forced choices).

## Known-answer arm (n=10, oracle-scored)

`decision_correct` against the sealed `correct_call`:

| arm | score | notes |
|----|----|----|
| Codex solo | **9 / 10** | wrong on c09 (said REPEATABLE READ makes the oversell safe; it does not) |
| Grok solo | **10 / 10** | correct on c09 |
| A_solo (better of the two, per decision) | **10 / 10** | grok carries c09 |
| A_2seat (two seats, no debate) | 9/10 agreed+correct; **c09 = unresolved split** | no consensus on c09 |
| **A_panel (Round 1 where disputed)** | **10 / 10** | c09 resolved correctly by Round 1 |

Seats agreed on 9/10; **c09 was the only disagreement.** There, Codex (both solo
and seat) said "**YES**, REPEATABLE READ prevents the oversell"; Grok said "**NO**"
with the decisive mechanism (in Postgres the loser gets a `40001` serialization
abort that must be caught and the *whole* transaction retried - it does not
transparently reserve stock; the real fix is an atomic conditional
`UPDATE ... WHERE stock > 0 RETURNING`). In **Round 1, Codex revised YES → NO**
("the answer to the brief... is NO"); Grok held. The panel converged on the
**correct** call. This is the first clean instance in any of the studies of Round
1 catching and correcting a confident solo error. (Full transcript:
[`results-raw/c09/`](results-raw/c09/).)

**What c09 does and does not show:**

- **Round 1 adds real value over the no-debate arm** (`A_panel − A_2seat` on c09:
  correct vs unresolved). The debate is what turned a split into the right answer.
- **The panel did NOT beat the best single model.** Grok-solo alone was already
  correct on c09. The panel only beat the *weaker fixed vendor* (Codex). Since you
  cannot know in advance which vendor is right on a given decision, the honest
  value is a **hedge against vendor choice** - the panel is `≥` any fixed single
  vendor (it matched Grok, it fixed Codex) - not an accuracy gain over
  best-of-breed.

## Ambiguous arm (n=10, independent judge)

On all 10 decisions the two seats **agreed** (no Round 1). Judged solo memo vs a mechanically-assembled
panel memo, blind, by gpt-oss (both metrics):

- **Aggregate rubric score: solo 118, panel 103 - solo wins.**
- **Forced choice: solo 9, tie 1, panel 0.**

**Caveat (stated):** the panel memo here was a *mechanical concatenation* of the
two seats' verdicts (per the protocol's no-orchestrator-prose rule), which reads
as more verbose than the tight solo memo and was penalized on the
padding/tightness criterion. This is the mirror of pilot-02, where an
orchestrator-*polished* panel memo may have flattered the panel. That both
renderings - crude-mechanical here, hand-polished there - still land at **solo ≥
panel on aggregate** is the robust takeaway; the exact margin is
rendering-sensitive and should not be over-read.

## Primary endpoint & decision rule

- **Pre-registered primary endpoint: mean `A_panel − A_2seat` (the value of Round
  1) on aggregate score.** `PROTOCOL.md` names this, and its decision rule was
  "declare lift only if this is positive and holds under both judges." It is
  effectively **untestable here**: seats disagreed on only 1 of 20 decisions, so
  Round 1 ran once; on that one (c09) it was positive (correct vs an unresolved
  split), and on the other 19 it was zero by construction (early stop). So the
  pre-registered lift criterion is **not met** - not refuted, but resting on a
  single data point, which is a real shortfall of this run as a *confirmatory*
  test (see Limitations). (An earlier version of this file wrongly labeled
  `A_panel − A_solo` as the pre-registered primary; corrected.)
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
  protocol's **non-OpenAI second judge (Meta/Alibaba) was not run** - left to
  replicators (the repo is frozen). So the two-judge confirmation is a replication
  target, and this result stands under a single independent judge.
- **The raw-vs-assembled panel-rendering control was not run**, and
  `must_catch_rate` was not tabulated per arm. Raw arm transcripts for all 20
  decisions are published under [`results-raw/`](results-raw/) so anyone can
  re-score. The ambiguous panel memo was also crudely rendered (mechanical
  concat), which handicaps it on tightness.

## Bottom line

At n=20, with a judge independent of the seats and orchestrator (single vendor),
oracle-scored known-answer decisions, and component-isolating arms: **Tribunal
shows no accuracy lift over a single strong model.** Its engine (disagreement →
Round 1) activated on 1 of 20 decisions; when it did, it correctly adjudicated
(n=1). The defensible value the data supports is narrow: **on decidable calls a
hedge that matched or beat any fixed single vendor and caught the wrong one in the
single case they diverged; on ambiguous calls, no better (judged worse) than the
solo memo** - plus a preserved counter-case and a discriminating test on contested
calls, paid for with a substantial operational cost. That is what the README now
claims, and no more.
