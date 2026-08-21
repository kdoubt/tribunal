# Tribunal validation - ablation protocol (pre-registered)

**Status: pre-registration.** This document and the sealed decision set in
[`decisions/`](decisions/) are committed **before** any arm is run, so results
cannot be curated after the fact. The commit that adds this file is the
timestamp; the commit(s) that add `results/` come later. If a later commit
changes an arm definition, a decision brief, or the scoring rubric, that is
called out explicitly in `CHANGELOG.md` and the affected runs are re-done.

## The question this answers

Tribunal's core premise is that a **panel of heterogeneous vendors catches
errors a single strong model would confidently miss, and that cross-examination
(Round 1) adds value over a cheap cross-vendor check.** Three independent
reviews correctly flagged that the repo demonstrates the process *works
mechanically* but does not yet show it produces *lift*. This ablation measures
the lift - or its absence - directly, and reports the absence honestly if that
is what the data shows.

## Arms (each decision is run through all four)

| Arm | What it is | Represents |
|----|----|----|
| **A0** | One seat (Codex), single shot, no oracle, no second opinion. | the naive solo baseline |
| **A1** | One seat (Codex), same task but explicitly told it may consult the oracle, then decide. | "one strong model + tools" - the real competitor |
| **A2** | Two isolated seats (Codex + Grok), independent Round 0, take agreement/union. **No Round 1, no cross-examination.** | a cheap cross-vendor sanity check |
| **A3** | Full Tribunal: Codex + Grok Round 0 → ledger → verbatim-relay Round 1 → oracle on checkable claims → bucketed verdict. | the method under test |

Seats are the same two vendors across arms. Every arm sees the **same brief**
(the `decisions/dNN-brief.md` file) and **never** sees the sealed ground truth.

## Ground truth

Each decision has a sealed rubric (`decisions/dNN-truth.md`) fixing, before any
run:

- `correct_call` - the right decision, backed by a **named authoritative
  oracle** (RFC / vendor docs / spec / a reproducible test), not the author's
  opinion.
- `must_catch` - the specific risks/facts a competent answer must surface.
- `landmine` - the plausible wrong answer a confident single model may give.

A decision is admissible only if `correct_call` is defensible from its oracle by
inspection. Decisions span a difficulty range on purpose: some are well-known
(controls, where all arms should agree and be correct - the expected finding is
"panel adds cost, no lift"); some are subtle (where solo may slip).

## Scoring (per decision, per arm)

Scored against the sealed rubric from the arm's **raw output**, which is saved
verbatim so anyone can re-score:

- `decision_correct` - reached `correct_call`? (0 / 0.5 partial / 1)
- `must_catch_rate` - fraction of `must_catch` items surfaced.
- `false_objections` - count of confident wrong claims, or correct things
  flagged as blockers (the panel's characteristic failure mode - a panel that
  "catches" more by objecting to everything is not better).
- `cost` - CLI invocations and rough wall-clock; A3 is expected to cost the
  most, so lift must be worth it.

To limit author bias, scoring is done by an **independent scorer** - a third
model (different from the two seats, given only the arm output + the sealed
rubric) - and the raw outputs are published so the score is auditable. Where the
oracle is mechanical (a test, a doc quote), it settles `decision_correct`
directly.

## The comparisons that matter

- **A3 − A1**: does debate beat one model with the same oracle access? This is
  the load-bearing claim. If ≤ 0 across the set, the method does not earn its
  cost and the README must say so.
- **A3 − A2**: does Round 1 cross-examination beat a no-debate cross-vendor
  check? Isolates the value of the *debate*, not just the *second vendor*.
- **A2 − A1**: does a second vendor alone (no debate) already capture most of
  the lift? If most lift is here, the honest recommendation is "use two seats,
  skip Round 1 unless disputed" - which the method already allows.
- **false_objections by arm**: the cost side of the ledger.

## Honesty commitments

- The decision set is fixed before running; adding decisions later starts a new,
  separately-labeled batch.
- **Every** decision's result is published, including panel losses, ties, and
  false objections - no decision is dropped for being unflattering.
- `n` is stated plainly. A pilot of a handful of decisions is a pilot, not
  proof; ~20+ with real-world outcomes is the bar for "validated" (per the
  external reviews). This protocol scales to that; it does not pretend a pilot
  reaches it.
