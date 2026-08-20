# ACH Brief - [the decision, framed as competing hypotheses]

<!-- Analysis-of-Competing-Hypotheses mode (see METHODOLOGY,
     "Competing-hypotheses mode (ACH)"). Use when the decision is "which
     explanation is right?", not "is this claim true?". Same isolation /
     ledger / verdict machinery as a normal panel; only Round 0's shape
     changes. Freeze this brief, then hand each seat the identical copy in
     isolation - all the CONTRACT rules still apply. -->

## Hypotheses (the set to weigh)

<!-- List every plausible explanation. Keep the H0 row so the matrix can never
     force a false choice between a bad set. -->

- H1:
- H2:
- H3:
- H0: none of these / something else / unknown

## Question & decision criteria

<!-- What this resolves, and what a good answer optimizes for (speed of fix,
     blast radius, reversibility, ...). If you want ties broken mechanically,
     state the rule here; otherwise unresolved ties go to you. -->

## Artifact(s)

<!-- Same rules as the standard brief: repo-relative paths or sanitized URLs,
     never secrets/usernames/private hosts. Seats read these directly. -->

-

## Seat output contract (Round 0, in isolation)

1. **Evidence list** (cap ~8). Number each item (`E1..`) with a DECISIVE
   pointer - `file:line`, a verbatim span, or an oracle invocation - not an
   argument.
2. **Matrix.** For each `Ei × Hj`, mark `C` (consistent) / `I` (inconsistent)
   / `NA` (not applicable). Actively look for **I**: an item consistent with
   every hypothesis is non-diagnostic - mark it and move on.
3. **Inconsistency claims** (this is what lands in the ledger; cap 6, same as
   a normal brief). Write one claim block per `I` cell you assert:

   ```
   CLAIM: E<i> is inconsistent with H<j> because <one sentence>
   EVIDENCE: the E<i> pointer (file:line / span / oracle) - or ASSUMPTION /
             SPECULATIVE / EXTERNAL+source if it isn't a direct read
   CONFIDENCE: 0-1 (scored later; calibrate)
   FALSIFIER: the observation that would make E<i> consistent with H<j>,
              or show E<i> itself is false
   ```

   A disputed *consistency* (you marked `C`, another seat marked `I`) is also
   a claim - write it the same way.
4. **Read.** Rank the hypotheses by **fewest inconsistencies, counting only
   hypotheses that have at least one diagnostic (non-`NA`) cell** - a
   hypothesis nothing was aimed at does not win by default; call it
   "untested / no stable position". Weigh severity: one decisive refutation
   outweighs several nits. Name the single cheapest piece of evidence that
   would most change the ranking - the discriminating test.
5. **VERDICT INPUT.** Your one-line most-likely hypothesis, plus the one(s)
   you cannot yet rule out.

A hypothesis wins by surviving refutation, not by collecting consistent
evidence. Do not confirm your way to an answer, and do not let `H0` (or any
thinly-tested hypothesis) win merely because nothing was aimed at it.
