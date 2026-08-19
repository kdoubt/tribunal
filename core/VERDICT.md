# The Verdict

Adjudication, not consensus. The verdict is written by the orchestrator
wearing only the adjudicate hat: it buckets what the panel produced; it MUST
NOT introduce new arguments. Fillable template: `templates/verdict.md`.

## Three buckets, in this order

1. **Independent agreement** - claims with status `agreed-r0`: made by all
   seats in Round 0, *before any cross-exposure*. These are the only
   findings that may be phrased as "the panel concludes…". Never
   vote-count anything else.
2. **Resolved after Round 0** - two labeled sublists, never mixed (a
   concession is not an oracle fact):
   - *Oracle-settled* (`verified`): the check named ("verified by running
     the test suite", "confirmed against the upstream changelog").
   - *Cross-examination-settled* (`conceded` / `overturned`): who yielded
     or withdrew, with a pointer to the Round 1 passage.
3. **Surviving dissent** - named claims, whose they are, and the *cheapest
   discriminating test* for each. A synthesis containing zero surviving
   objections on a genuinely contested question should be treated as
   suspect.

Claims still `open` (uncontested, nothing turned on them) belong in none of
the buckets: they are listed in the verdict's Record as **unexamined, not
endorsed**.

## The recommendation

Mode-tagged, built **only** from bucketed claims:

| Mode | Meaning |
|---|---|
| `ship` | Proceed; buckets 1-2 support it and no surviving dissent is decision-relevant |
| `dont` | Do not proceed; grounds in buckets 1-2 |
| `decide-after-check` | The named discriminating test settles it; run that first |
| `human-call` | Surviving dissent is decision-relevant and not cheaply checkable; the human decides, with the dispute framed for them |

Stopping early because Round 0 already agreed skips cross-examination but
still maps the `agreed-r0` claims onto whichever mode they support -
including `dont`, `decide-after-check`, or `human-call`; early agreement is
not automatically `ship`.

If Round 0 conflicted and cross-examination plus oracles didn't resolve a
decision-relevant claim, the orchestrator does not pick a winner by fiat: it
either runs the discriminating test or hands the human a framed decision.

**Pre-delegation exception:** the human MAY write explicit decision criteria
into the frozen brief up front; the orchestrator then applies those criteria
mechanically to surviving dissent and says so in the verdict.

## Preserve uncertainty

State which claims were *verified* versus merely *agreed*. Don't launder
panel opinions into facts. If the evidence does not resolve a dispute, say
so - pretending agreement destroys exactly the information the panel exists
to produce.
