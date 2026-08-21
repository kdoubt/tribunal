# Verdict - repo-monorepo-vs-polyrepo

**Mode: surviving dissent (decide-after-check).** After a full Round 1
cross-examination, the two seats **held opposing verdicts** - Seat A: monorepo
(A); Seat B: polyrepo (B). The method does **not** average them into a hedge.
It states exactly what both seats agreed on, isolates the one dispute that
survived, and routes that dispute to the cheapest test that would settle it -
here, the same test **both seats independently proposed.**

> This was a **role-incentivized stress-test**: the orchestrator assigned Seat A
> to champion the monorepo and Seat B the polyrepo (roles as incentives, not
> personas - two real, different-vendor CLIs citing real sources). So the
> disagreement is *engineered*, not organic. That is deliberate - it is how you
> manufacture a hard Round 1 on a genuinely no-consensus question and watch what
> survives adversarial pressure. Contrast the sibling example
> [`../api-auth-jwt-vs-sessions/`](../api-auth-jwt-vs-sessions/), where two
> *neutral* seats agreed in isolation.

## Common ground (both seats, after cross-examination)

Promoted to the panel's conclusion - these are not in dispute:

- **Atomic cross-project commits are a genuine monorepo advantage**, and are the
  **less-frequent** case for *loosely related* services (agreed A1 ≈ B5).
- **Nx affected-graph + caching is a real mechanism**, not vapor (B conceded).
- **Polyrepo has stronger repository-level isolation by default** (A conceded).
- **Blast radius is a tradeoff, not a win for either side** (agreed A5 ≈ B3):
  monorepo makes downstream impact visible *pre-merge* but puts one `main`/CI in
  everyone's path; polyrepo defers breakage to an *opt-in* version bump but
  accrues drift.
- **Whichever is chosen, its day-one risk must be funded up front:** for
  monorepo, affected-only CI + caching + dependency boundaries + path ownership +
  independent deploy (the unified-CI **choke point**); for polyrepo, an internal
  registry + Renovate/Dependabot + contract tests (**version drift**).

## Surviving dissent (the one dispute Round 1 did not resolve)

**Can *this specific org* - ~25→60 engineers, 8+ loosely related polyglot
services, explicitly *not* Google/Meta-scale and constrained *not* to build a
bespoke build system - operate a monorepo whose affected-CI stays fast and whose
`main` stays green, *without* standing up a dedicated build-platform team?**

- **Seat A (monorepo):** yes - this is attainable *configuration* (Nx affected +
  remote cache on GitHub Actions), and the polyrepo "commodity" path is itself a
  platform (templates, registry ops, dependency automation, contract testing,
  fleet-wide policy). *A2 held at 0.88.*
- **Seat B (polyrepo):** no - a *working* monorepo here is an owned project graph
  + remote cache + merge-queue + a CI-owning team answering Nx's seven
  organizational questions; on a polyglot estate that is more than "turn on Nx,"
  and it is exactly the standing infrastructure the brief says the company may
  not be able to staff. *B2 revised 0.90→0.85, held.*

Neither seat overturned the other. Both revised toward the middle (A conceded
isolation-by-default and a catalog's discovery win; B conceded pre-merge
visibility and that Nx-affected is real) but **each kept its verdict.** This is a
real, load-bearing disagreement about *this org's operational capacity*, not a
factual error on either side - so it is recorded as surviving dissent rather than
resolved by debate.

## Recommendation: decide after the check (do not pick blind)

Because the dissent turns on a **measurable** property of this specific
codebase and team, the panel does not flip a coin. It runs the cheapest
discriminating test first. **Both seats independently proposed essentially the
same one:**

- Seat A: *"a two-week monorepo pilot containing one shared library and two
  representative services, measuring affected-CI time, unrelated failures, and
  effort for one cross-cutting change against the current polyrepo workflow."*
- Seat B: *"two-week spike of two services + one shared lib in one GitHub repo
  with Nx affected and CODEOWNERS; if median PR CI stays within ~20% of today's
  per-repo CI and unrelated-red hours stay near zero **with no dedicated owner**,
  A's Claim 2 stands."*

### The oracle (pre-registered pass/fail)

Run the **two-week spike**: 1 shared library + 2 representative services in one
repo, Nx affected + CODEOWNERS, on unmodified GitHub Actions, **staffed by no
dedicated build-platform person.** Adopt **monorepo (A)** iff *all* hold:

1. **Median PR CI time** stays within **~20%** of today's per-repo CI.
2. **Unrelated-red hours** (a team blocked on another team's failure/flake on the
   shared `main`) stay **near zero**.
3. The one **cross-cutting change** is meaningfully cheaper than the equivalent
   polyrepo publish-and-bump.

If checks 1-2 fail without a dedicated owner, **Seat B is right for this org -
choose polyrepo (B)** and fund the registry + Renovate + contract-test stack.

### Second, cheaper reading that sets the weight

Sample **~3 months of merged PRs** and measure the fraction that touch **≥2
projects.** This directly weighs the converged D1 dispute: a high fraction pushes
toward A (atomic commits earn their keep); a low fraction confirms both seats'
read that cross-cutting is "occasional" and tilts toward B. This is a
read-only analysis - run it first; it is nearly free and may make the spike's
result obvious in advance.

## What the panel explicitly did NOT do

- It did not average A and B into "start polyrepo, add a monorepo later" or any
  other unearned hybrid. The brief forbade "do both," and neither seat's evidence
  supported a blend - so the honest output is *a decision procedure*, not a
  mushy midpoint.
- It did not promote either seat's *single-sourced* framing (DORA-dominance for
  B, onboarding-default for A) to consensus; those stayed with their authoring
  seat, discounted by the other's rebuttal.
