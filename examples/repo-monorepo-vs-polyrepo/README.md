# Example run - monorepo vs polyrepo (role-incentivized, surviving dissent)

A **real** Tribunal panel on a genuinely no-consensus engineering decision, run
as a **role-incentivized stress-test** and published in the current template
format. This is the deliberate counterpart to
[`../api-auth-jwt-vs-sessions/`](../api-auth-jwt-vs-sessions/): there, two
*neutral* seats independently agreed and the panel honestly stopped at Round 0;
**here, the seats were assigned opposing sides, Round 1 ran, and dissent
survived into the verdict.**

## Read this first: what "role-incentivized" means

The two seats did **not** organically disagree. The orchestrator **assigned**
Seat A to champion the monorepo and Seat B to champion the polyrepo. Crucially:

- **Roles are incentives, not personas.** Each seat is a real, different-vendor
  CLI (**Codex CLI** and **Grok CLI**) citing real, checkable public sources. The
  role only decides *which side each argues hardest* - it does not make a seat
  invent evidence or role-play a character. Seats were explicitly told they may
  concede genuine tradeoffs and must not fabricate.
- **Why do this at all?** On a genuinely balanced question, neutral seats often
  converge (see the sibling example) - which is trustworthy but doesn't exercise
  cross-examination. Assigning opposing incentives **manufactures a hard Round 1
  on purpose**, so you can watch which claims survive adversarial pressure and
  which fold. It is a stress-test of the *arguments*, not a poll of opinion.
- **So the disagreement is engineered.** Do not read "the seats disagreed" here
  as evidence the field is split (though on this question it genuinely is). Read
  it as: *under maximal adversarial pressure from both directions, this is what
  held.*

The role assignment is stated openly at the bottom of [`brief.md`](brief.md) and
re-flagged in every seat file and in [`verdict.md`](verdict.md).

## The decision

A B2B company (~25→60 engineers, 8+ loosely related polyglot services, *not*
Google-scale, *cannot* build a bespoke build system) must standardize now on
**one monorepo** or **polyrepo** for all product code, weighed on cross-cutting
ease, team autonomy, CI cost, and blast radius. Full brief in
[`brief.md`](brief.md).

## What actually happened

- **[Round 0](round0-seat-a-codex.md)** (isolated, verbatim): Seat A built a
  five-claim monorepo case (atomic commits; Nx-affected CI is attainable, not
  bespoke; CODEOWNERS ownership; onboarding; pre-merge impact visibility). Seat B
  built a six-claim polyrepo case (DORA loosely-coupled autonomy; a *working*
  monorepo is a platform product + a dedicated team; versioned-dep blast-radius
  containment; the consensus tax; atomic commits are the "correct loss";
  reversibility).
- **[Round 1](round1-seat-a-codex.md)** (each seat cross-examined the other's
  claims): both **conceded real points** - A conceded polyrepo's stronger
  isolation-by-default and a catalog's discovery win and revised two claims
  *down*; B conceded that Nx-affected is a real mechanism and that pre-merge
  visibility is genuine, and revised two claims down. **Neither verdict was
  overturned.**
- **[The ledger](ledger.md)** tracks all 11 claims, the six dispute axes, and
  which Round 1 *converged* (atomic-change value, blast-radius, onboarding) vs
  which *survived*.
- **[The verdict](verdict.md)** records **surviving dissent** on the load-bearing
  question - *can this specific org run a monorepo without a bespoke build system
  or a dedicated build-platform team?* - and refuses to average it. Instead it
  routes the dissent to the cheapest discriminating test. Remarkably, **both
  opposing seats independently proposed nearly the same test** (a ~two-week
  monorepo spike, 1 shared lib + 2 services, measuring affected-CI time and
  unrelated-red hours with no dedicated owner), so the verdict adopts the
  intersection of their two falsifiers with pre-registered pass/fail.

## What this example demonstrates

- **Round 1 cross-examination done right** - verbatim relay of the opposing
  claims, structured ATTACK / CONCEDE / REVISE, real concessions and confidence
  revisions on both sides.
- **Surviving dissent, honestly reported** - the method preserves a genuine
  disagreement instead of smoothing it into a hedge, and it names *exactly what
  the disagreement depends on* (this org's operational capacity), which is
  measurable.
- **Convergent falsifiers as an oracle** - when two adversarial seats name the
  same settling test, their intersection is a stronger oracle than either seat's
  confidence; the verdict pre-registers it.
- **No unearned hybrid** - the brief forbade "do both," and the panel produces a
  *decision procedure* (run the test, then commit), not a mushy midpoint.
- **Contrast with the sibling example** - `api-auth-jwt-vs-sessions/` shows
  independent agreement + an honest empty dissent bucket; this shows engineered
  disagreement + a surviving dissent bucket. Together they cover both ends of the
  method.

## Provenance & sanitization

Run 2026-08-21 with the two CLIs then current. Seat outputs are verbatim; one
leading tool-narration line was trimmed from each Grok file (noted in-file), and
nothing in the arguments was edited. The decision is generic and the seats cited
only public sources (DORA, Google's CACM monorepo paper, Netflix, Nx, Backstage,
Bazel, GitHub CODEOWNERS, AWS, and a published monorepo literature review) - no
personal or private data appears. Citations were **not** independently
fact-checked in-panel; this run demonstrates the *method*, not a citation audit.
As a demonstration run, the retro's T1 (real outcome) is illustrative rather than
a shipped result.
