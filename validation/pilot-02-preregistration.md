# Pilot 02 - pre-registration (ambiguous, no-ground-truth decisions)

**Status: pre-registration**, committed before any arm or judge is run. Motivated
by [pilot-01](results/pilot-01.md)'s null: known-answer decisions are the wrong
instrument because strong models already decide them solo, *and* they are the
category Tribunal's own scope says not to convene a panel for. Pilot-02 tests the
value the method actually claims - on **ambiguous decisions with no knowable
ground truth**, does the panel produce a more decision-useful result than the
same strong model alone?

## Arms (per decision)

| Arm | What it is |
|----|----|
| **A1** | **Codex alone** writes a decision memo (call + key risks + single biggest risk + a recommended discriminating test). One strong model, one shot. |
| **A3** | **Full Tribunal**: Codex + Grok isolated Round 0 → ledger → verbatim-relay Round 1 (these ambiguous briefs are *expected* to produce disagreement) → verdict with buckets + surviving dissent + the cheapest discriminating test. The verdict is A3's "memo." |

The **same Codex model** is in both arms, so the A3−A1 delta isolates *what the
panel adds on top of one strong model* (the second vendor + cross-examination +
the verdict structure), not raw model strength.

## No ground truth - so scoring is by a blind judge

These decisions have no single right answer (excellent engineers disagree). We do
**not** score accuracy. We score **decision usefulness**, blind:

- The two memos for a decision are stripped of any arm label, given neutral names
  (Memo X / Memo Y), and their **order randomized per decision** (fixed in advance
  in the run log).
- **Judge = Claude CLI** - the third vendor, *not* one of the two seats. It never
  learns which memo is solo vs panel. Limitation, stated plainly: Claude is also
  the orchestrator's vendor; to contain that, the judge is a separate blind
  invocation and **all memos are published** in `results/` so anyone can
  re-judge with any model. A second independent judge may be added later.

### Blind-judge rubric (per memo, 0-3 each)

1. **Risks surfaced** - did it identify the decision's real tradeoffs and failure
   modes? (0 none / 3 the load-bearing ones).
2. **Discrimination** - did it name a concrete, cheap test or criterion that would
   actually settle the call? (0 none / 3 a sharp pre-registerable test).
3. **Honesty & decisiveness** - did it commit *and* mark what's genuinely
   uncertain, rather than waffle or false-confidence? (0 / 3).
4. **Freedom from false objections / padding** - penalize flagging non-issues as
   blockers and verbose hedging. (0 badly padded / 3 tight). *This is the panel's
   characteristic failure mode and the control against "more text = better."*

Then a forced choice: **which memo would you rather hand a team making this
call - X, Y, or genuine tie -** with one sentence why.

## The comparison that matters

**A3 − A1** on total judged score and on the forced choice, across the set. If the
panel does not win (or ties while costing far more), the honest recommendation is
"use a strong solo model for ambiguous calls too" - and the README will say so.

Secondary read: does A3 win specifically on **Discrimination** (naming the
resolving test) and **Risks surfaced**, which is where the method's dissent
structure should help, while not losing on **false objections/padding** (where a
two-seat + Round 1 process is most at risk of bloat)?

## Honesty commitments (same as pilot-01)

- Decisions fixed before running; the judge is blind; every decision's result is
  published including panel losses and ties.
- Padding is penalized so the panel can't win on verbosity.
- `n` stated plainly; this is a pilot, not proof. The bar for "validated" remains
  ~20+ decisions and, ideally, real decisions whose outcomes later become known.
