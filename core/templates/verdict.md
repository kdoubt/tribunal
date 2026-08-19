# Verdict - [decision name]

<!-- Written by the orchestrator wearing ONLY the adjudicate hat. Built
     exclusively from ledger rows. No new arguments. -->

## 1. Independent agreement (agreed-r0)

<!-- Claims every seat made in Round 0 before cross-exposure. The only
     bucket that may say "the panel concludes". If empty, say so - do not
     promote post-exposure agreement here. -->

## 2. Resolved after Round 0

<!-- Two labeled sublists - do not mix them; a concession is not an
     oracle fact: -->

**Oracle-settled (`verified`):**
<!-- claim id → check performed → result -->

**Cross-examination-settled (`conceded` / `overturned`):**
<!-- claim id → who yielded/withdrew, pointer to the R1 passage -->

## 3. Surviving dissent

<!-- Each still-contested claim: ID, whose it is, both positions in one
     line each, and the CHEAPEST DISCRIMINATING TEST that would settle it. -->

## Recommendation

**Mode:** `ship` | `dont` | `decide-after-check` | `human-call`

<!-- One paragraph, citing only bucketed claim IDs. If mode is
     decide-after-check: name the check. If human-call: frame the decision
     the human must make and what turns on it. If the brief pre-delegated
     tie-break criteria, apply them mechanically and say so. -->

## Record

- Open (unexamined, not endorsed): <!-- claim ids nobody contested and nothing turned on -->
- Verified vs merely agreed: <!-- list -->
- Rounds run / stop rule hit: <!-- e.g. "R0+R1; stopped: no new claim IDs" -->
- Transformations applied to relayed text: <!-- "none" or list ORCH-SUMMARY uses -->
