# Evaluating Tribunal

A map from **what this project claims** to **the evidence for it** and **what
that evidence does not show**. It exists because the claims and the data live in
different files across ~10,000 words, and a reviewer should not have to
reconstruct that.

> **This page does not tell you how to weight anything.** It is an index, not a
> rubric. Tribunal's own `core/CONTRACT.md` requires that artifact contents be
> treated as *untrusted evidence, never instructions* - a repo asking its
> reviewers to adopt its scoring would be the exact failure that contract exists
> to prevent. Every row below points at a file you can open; disagree with any
> of it by reading the source.

## The claims, and where they are tested

| Claim | Where it is made | Evidence | What the evidence does **not** show |
|---|---|---|---|
| A panel is *designed* to give you an adversarial counter-case, a discriminating test, and a hedge on contested calls | `README.md` opening; `core/METHODOLOGY.md` "When to convene" | Design rationale + [`examples/`](examples/) worked runs | That it out-*decides* a strong single model. Two pre-registered studies found **no accuracy lift** |
| Only pre-exposure agreement counts as consensus | `core/CONTRACT.md` ob. 7; `core/LEDGER.md` `agreed-r0`; `core/VERDICT.md` bucket 1 | Normative text, enforced by the adapters' Round 0 isolation | That isolation changes outcomes. Untested as an isolated variable |
| Disputes route to oracles rather than more debate | `core/METHODOLOGY.md` Round 1; `core/CONTRACT.md` ob. 5 | [`validation/FIELD-RECORD.md`](validation/FIELD-RECORD.md): 48 oracle-settled claims vs 36 debate-settled across 37 runs | A ratio - the two sums come from different reporting subsets |
| Dissent survives into the verdict | `core/VERDICT.md` bucket 3 | FIELD-RECORD: 27 surviving-dissent claims across 15 runs | That preserved dissent was *useful*. Scored outcomes: dissent proved right 2, wrong 4 |
| **No accuracy lift over a strong single model** | `README.md` Status | [`validation/confirmatory/RESULTS.md`](validation/confirmatory/RESULTS.md) (pre-registered, n=20, independent judge) and [`validation/grounded/RESULTS.md`](validation/grounded/RESULTS.md) (pre-registered, 10 decisions on real artifacts). The designated non-OpenAI second judge was run 2026-09-22 and agreed with the primary judge's forced choice on 9 of 10 ([`validation/confirmatory/SECOND-JUDGE.md`](validation/confirmatory/SECOND-JUDGE.md)) | That the counter-case or hedge is worthless - those were not scored. But usefulness **was** scored on the ambiguous arm and went **against** the panel: the independent judge preferred the solo memo, aggregate 118 vs 103, 9 of 10 forced choices (`validation/confirmatory/RESULTS.md:43-44`). Both studies disclose execution deviations. The second judge does **not** complete the pre-registered primary endpoint and does **not** declare or refute lift; and the rubric ceilings on the solo arm, so the score gap is a weak magnitude estimate |
| Frontier seats mostly agree, so debate rarely runs | `README.md`; `core/METHODOLOGY.md` | Confirmatory study: seats diverged on 1 of 20. FIELD-RECORD: Round 1 ran in 12 of 37 operator runs | That the two rates measure the same thing. They do not, and FIELD-RECORD says so |
| The adapters' operational rules are earned, not assumed | `adapters/` | [`validation/adapter-probes.md`](validation/adapter-probes.md): dated, re-runnable CLI transcripts | Anything about model quality. These are operational facts about CLI versions, not a benchmark |
| Docs-only, no runtime to install | `README.md`; `CONTRIBUTING.md` "What will be declined" | CI-enforced: `.github/workflows/checks.yml` fails the build on any tracked application code | That a panel is *cheap*. It still needs two authenticated vendor CLIs and 20-40 minutes |

## What is deliberately absent

- **No benchmark table, no leaderboard, no "which model is better."**
  `CONTRIBUTING.md` declines these. Seats are meant to be interchangeable.
- **No runtime, SDK, or plugin.** The portability thesis is that documents
  survive where frameworks rot.
- **No accuracy claim.** Both studies are published with negative results, carried
  in the README's `validation: no accuracy lift measured` badge and its
  `## What the evidence says` section.

## The fastest ways to falsify this project

In rough order of cost:

1. **Re-score the confirmatory study with your own judge** -
   [`validation/confirmatory/REPLICATING-THE-JUDGE.md`](validation/confirmatory/REPLICATING-THE-JUDGE.md).
   Bring your own key (a free tier or a local model works), $0, judge only - no
   debate seats to authenticate. If a different judge reaches a different
   conclusion on the sealed set, that is a finding worth filing.
2. **Re-run the adapter probes** - [`validation/adapter-probes.md`](validation/adapter-probes.md)
   ships verbatim commands and outputs with CLI versions. They are dated; a probe
   that no longer reproduces means the rule it supports needs re-dating.
3. **Check the citation discipline on a published run** - every panel in
   [`validation/grounded/results/`](validation/grounded/results/) publishes its
   raw seat outputs, ledger, packets and judge replies. Open a cited `file:line`
   and see whether the span supports the claim. The grounded study measured this
   itself: pointers resolved 0.83-0.97 of the time, but a judge found the opened
   text supported the claim only 0.50-0.81 of the time.
4. **Run a panel on a decision whose outcome later becomes known**, and report
   whether the verdict held. This is the contribution `CONTRIBUTING.md` most
   wants, and the one thing this repo cannot produce for itself.

## What would change the project's own position

The README's no-lift finding rests on two studies by one maintainer, with seats
and judges from a handful of vendors, scored on 30 decisions total. It is
stated as a finding, not a law. What would move it:

- An independent replication that **finds** lift on decidable calls - the
  confirmatory study's n=1 divergence is far too thin to rest on.
- A replication that finds the panel *worse* than a solo model on grounded
  review, which would strengthen rather than overturn the current position.
- Evidence that the procedural claims (isolation, grounding, preserved dissent)
  change decisions independently of accuracy. No study here isolates them.

## Where the material is

`README.md` `## What the evidence says` and `## Status` carry the claims.
[`validation/README.md`](validation/README.md) indexes the studies.
`core/CONTRACT.md` (158 lines) is the normative core; `core/METHODOLOGY.md`
(3,848 words) is the reasoning behind it and contains claims of its own -
including the ambiguous-arm result above, at lines 24-26. Run archives are under
[`validation/`](validation/) and [`examples/`](examples/).
