# Pilot 01 - results (n=5, known-answer decisions)

**Run:** 2026-08-21. **Seats:** Codex CLI (GPT-5-class), Grok CLI (Grok-4-class).
**Decisions:** [`../decisions/`](../decisions/) d1-d5, pre-registered before this run.
**Raw arm outputs:** [`pilot-01/`](pilot-01/) (verbatim, so anyone can re-score).

## Headline

**No lift, and the test could not even exercise Round 1.** Every arm reached the
`correct_call` on every decision. The two seats never disagreed, so stop rule
(a) fired on all five and **no Round 1 ran** - the panel collapsed to its
two-isolated-seats form (A2), which was no more accurate than a single model
(A0). This set was **too easy to discriminate**: current frontier models have
absorbed these classic traps, so a solo model gets them right *without* an
oracle, and there is nothing for a panel to add.

This is a null result, published in full per the protocol. It is **not**
evidence that panels are useless - it is evidence that *this decision set cannot
measure panel lift*, plus one real operational failure (below).

## Scorecard

`decision_correct`: 1 = reached the sealed `correct_call`.

| decision | difficulty | A0 solo | A1 solo+oracle | A2 two-seat (no debate) | A3 full panel | seats disagreed? |
|----|----|:--:|:--:|:--:|:--:|:--:|
| d1 job-queue race | subtle | 1 | 1 | 1 | 1 (=A2, early stop) | no |
| d2 password hashing | control | 1 | 1 | 1 | 1 (=A2, early stop) | no |
| d3 webhook idempotency | subtle | 1 | 1 | —* | —* (no valid panel) | n/a (1 seat died) |
| d4 float money | control | 1 | 1 | 1 | 1 (=A2, early stop) | no |
| d5 invoice-number race | subtle | 1 | 1 | 1 | 1 (=A2, early stop) | no |

`must_catch` coverage was high across the board (e.g. every arm on d1 named the
SELECT/UPDATE race, `FOR UPDATE SKIP LOCKED`, and that an index alone doesn't fix
it; Grok on d1 caught the extra subtlety that the claim UPDATE lacks an
`AND status='pending'` re-check). **`false_objections`: 0** - no arm flagged a
correct design as broken or made a confident-wrong claim.

- **A3 − A1 (does debate beat one model + oracle?): 0 across the 4 evaluable
  decisions** (d3 excluded - no valid panel). Undetermined-and-zero: no dispute
  arose, so Round 1 never executed.
- **A3 − A2 (does Round 1 beat a cheap cross-vendor check?): not testable here** -
  every panel early-stopped at Round 0.
- **A2 − A1 (does a second vendor alone help?): 0 across the 4 evaluable
  decisions.**

## The one real failure: a dead seat

*d3, Grok seat.* Given the brief **inline** in the prompt, Grok nonetheless went
looking for a "webhook handler" file in the workspace, narrated three times, and
returned **no CLAIM/VERDICT** - a silent seat death (exit 0, narration only),
the exact failure mode `adapters/shell/README.md` documents. Solo Codex answered
d3 correctly regardless, so no accuracy was lost, but this is a live data point
for the reviewers' "operational friction" critique: **Grok's headless reliability
is a real cost of the two-vendor requirement.** On d3 there was **no valid panel**
(one of two required seats produced no position), so A2/A3 are scored `—` there,
not `1` - an earlier version marked them `1*`, which overstated it; the honest
statement is "solo got d3 right; the panel could not be evaluated."

## Corrections & limitations (added after a re-review)

Two frontier-model re-reviews (Codex, Claude) flagged reporting gaps in this
pilot; recording them here rather than papering over them:

- **Who scored:** the rubric was applied by the **orchestrator** (by inspection),
  **not** the independent third-model scorer `PROTOCOL.md` calls for. Practically
  low-risk here (all arms were unanimous and the rubric items are mechanically
  checkable against the sealed truth), but it is a deviation from the protocol and
  should be an independent scorer in any confirmatory run.
- **`must_catch_rate` was not tabulated per arm.** The write-up reports coverage
  as "high" with examples, not the pre-registered per-arm fraction. Read the raw
  outputs in [`pilot-01/`](pilot-01/) to re-score precisely.
- **The four arms do not isolate single mechanisms.** A1 may use an oracle, A2
  does not, A3 does - so A2-vs-A1 changes vendor count *and* oracle access, and
  A3-vs-A2 changes Round 1 *and* oracle access. This pilot compares *whole arms*
  (does the full panel beat a solo model), not isolated components; a clean
  component ablation would hold oracle access constant across arms.
- **d3 blinding was instruction-based, not access-separated.** The sealed truth
  files sat in the same workspace; one seat's output even narrates that it will
  "skip the truth file." That is evidence of intended compliance, not enforced
  blinding. A confirmatory run should keep truth files out of the seats' reachable
  filesystem entirely.

## What this means (honestly)

1. **The harness works and the pre-registration held.** Briefs were frozen,
   truth was sealed, arms never saw it, raw outputs are published. That
   machinery is sound and reusable.
2. **Known-answer engineering decisions are the wrong instrument for showing
   panel lift** - and, tellingly, they are the category Tribunal's own scope
   *tells you not to convene a panel for* ("skip anything a compiler, test, or
   grep can settle"). A strong model already decides them correctly; the panel
   adds cost and its early-stop rule correctly declines to run. So a clean null
   here is consistent with the method's own claims, not a refutation of them.
3. **The load-bearing claim - that Round 1 cross-examination catches a confident
   error - remains untested.** It needs decisions where either (a) a strong solo
   model genuinely, confidently errs (so a panel has an error to catch), or (b)
   the two seats actually disagree. Neither happened here.

## Next (pilot-02), pre-registered separately before running

Two candidate directions, to be chosen and pre-registered before any run:

- **Harder decidable set** - decisions where a strong model reliably slips:
  subtle spec/edge-case behavior, version-specific or post-training-cutoff
  gotchas, adversarially constructed "looks-right" traps. This is the direct
  test of "panel catches solo's confident error." Risk: hunting for
  solo-failures until one appears is p-hacking; mitigate by fixing the set and
  the base-rate reporting in advance.
- **Reframe to the actual value prop** - Tribunal claims value on *ambiguous,
  no-knowable-ground-truth* decisions (architecture, migrations), where the
  yardstick is not accuracy-vs-truth but **decision-relevant risks surfaced,
  false objections avoided, and whether a better discriminating test is named** -
  scored by a blind independent panel. This measures what the method actually
  claims, not a category it disclaims.

Neither is run yet. This file is the honest floor: on n=5 easy known-answer
decisions, the panel showed no lift, and one seat died.
