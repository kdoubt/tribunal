# Confirmatory study - results (n=20)

**Run:** 2026-08-21. **Seats:** Codex CLI (GPT-5-class), Grok CLI (Grok-4-class).
**Independent judge:** gpt-oss-120b (fourth vendor, temp 0). **Decisions:** the
pre-registered set (10 known-answer with sealed oracle truth; 10 ambiguous).
**Design:** [`PROTOCOL.md`](PROTOCOL.md), committed before the run. Raw arm
outputs for the decisive case are in [`results-raw/`](results-raw/).

## Headline

**No lift, confirmed - and the panel's engine almost never activates.**

- **Seats disagreed on 1 of 20 decisions.** On the other 19 the two vendors
  independently reached the same call, so Round 1 never ran and `A_panel = A_2seat`.
  This is the study's most robust, judge-independent finding, and it confirms the
  pilots at n=20: **frontier models mostly converge, so the panel machinery idles.**
- **On the 1 disagreement (c09), Round 1 worked** - it correctly adjudicated the
  split, moving the *wrong* seat to the right answer. So the mechanism is sound; it
  just rarely gets to act.
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

All 10 seats **agreed** (no Round 1). Judged solo memo vs a mechanically-assembled
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

- **Primary (`A_panel − A_2seat`, the value of Round 1):** only *testable* on the
  1 decision where seats disagreed (c09), where it was **positive** (correct vs
  unresolved). On 19/20 it was zero by construction (early stop). So Round 1's
  value is real but **rarely triggered** - the binding constraint is the 1/20
  disagreement rate, not Round 1's quality.
- **Decision rule outcome: lift is NOT declared.** The mean `A_panel − A_solo` is
  ≤ 0 (tie on known-answer, negative on ambiguous), and the panel does not beat a
  best-of-breed single model under the independent judge. The no-lift prior from
  the pilots **stands, now confirmed at n=20 with an independent fourth-vendor
  judge, oracle scoring, and component-isolating arms.**

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

## Limitations (honest)

- **n = 20; one judge vendor.** The independent judge is gpt-oss (OpenAI lineage,
  so not fully vendor-disjoint from the Codex seat, though Codex is in both arms
  so it largely cancels). The designated **non-OpenAI second judge (Meta/Alibaba)
  is not yet added** - the two-judge confirmation the protocol calls for is
  incomplete.
- **Ambiguous panel rendering was crude** (mechanical concat), which handicaps the
  panel on tightness; a fair rendering study (the protocol's raw-vs-assembled
  control) was not run.
- **`must_catch_rate` per arm was not tabulated** for known-answer; decision_correct
  is the reported metric there.
- **Only 1 decision exercised Round 1**, so the primary endpoint rests on a single
  data point. Powering the Round-1 comparison would need decisions selected to
  provoke disagreement (which biases the sample) or a much larger n.

## Bottom line

At n=20, with an independent fourth-vendor judge, oracle-scored known-answer
decisions, and component-isolating arms: **Tribunal shows no accuracy lift over a
single strong model.** Its engine (disagreement → Round 1) activated on 1 of 20
decisions; when it did, it correctly adjudicated. The defensible value the data
supports is narrow and real: **a hedge that is never worse than a fixed single
vendor and sometimes catches its error, plus a preserved counter-case and a
discriminating test on contested calls - paid for with a substantial operational
cost.** That is what the README now claims, and no more.
