# Verdict - what should the Pressure-Test methodology be?

**Mode:** `ship` (the synthesized methodology is `core/` of this repo)

## 1. Independent agreement (agreed-r0)

- The panel is a structured dispute whose value is uncorrelated error
  processes + explicit adjudication (A1/B-position).
- Round 0 must be isolated and parallel; post-exposure agreement is cheap;
  stop immediately if R0 agrees (B2/A-r1).
- Oracles (tests, compiler, primary docs) settle checkable claims; never
  debate a checkable fact (B4/A2-¶).

## 2. Resolved in cross-examination

- Round count: 2 debate rounds default, triggered 3rd, verification steps
  don't count as rounds (A2 overturned; B1 modified - both seats' final
  verdict inputs converged).
- Time-cost threshold for convening: overturned (B6) in favor of
  expected-loss factors (irreversibility, blast radius, observability).
- Claim ledger + provenance tags: adopted (A3/A4, conceded by B in R1).

## 3. Surviving dissent

| Dispute | A (Codex) | B (Grok) | Cheapest discriminating test |
|---|---|---|---|
| Attribution in relay | Blind during judgment where feasible | Attributed always | Run same panel blinded vs attributed; compare rebuttal quality and bias artifacts |
| Tie-breaking authority | Orchestrator decides via explicit criteria | Never; oracle or human | Audit verdicts over time for orchestrator-prior leakage |

**Adjudication applied** (criteria stated, per the anti-laundering rule):
attributed-by-default for two-seat panels because blinding is ineffective
there (style deanonymizes) - blinding remains a MAY at 3+ seats; the
orchestrator never breaks ties except under explicit pre-delegated criteria
in the frozen brief. Both calls are recorded in `core/CONTRACT.md` and
`core/VERDICT.md`, and the losing arguments are preserved here.
