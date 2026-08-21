# Claim Ledger - repo-monorepo-vs-polyrepo

The orchestrator's working state. Because this is a **role-incentivized** panel
(Seat A assigned to champion the monorepo, Seat B the polyrepo), the two seats
are on opposing sides *by construction* - so nearly every decision-relevant
claim is disputed and a Round 1 cross-examination was always going to run. That
is the point: this example exists to show Round 1 and a **surviving-dissent**
verdict, the mirror image of [`../api-auth-jwt-vs-sessions/`](../api-auth-jwt-vs-sessions/)
(where two *neutral* seats independently agreed and the panel stopped at Round 0).

Provenance tag on every row is `PANELIST-CLAIM (EXTERNAL)` - each claim carries a
named source or a concrete mechanism, not a vibe. Confidence is shown as
`R0 → R1` where the authoring seat revised it during cross-examination.

## Seat A claims (champion: monorepo)

| id | claim (compressed - full text in round0-seat-a-codex.md) | evidence | confidence | post-R1 status |
|----|-----|----|----|----|
| A1 | Atomic cross-project commits make the monorepo strongest for cross-cutting change/refactor. | Brito/Terra/Valente review; Google CACM | 0.96 | **converged** - both seats agree the mechanism is real *and* that it is rarer here ("loosely related / occasional"); decisive only if cross-cutting is frequent |
| A2 | At ~60 eng / 8+ projects, monorepo CI is an attainable *configuration* problem (Nx affected + cache), not a bespoke-build-system problem. | Nx affected docs | 0.88 (held) | **surviving dissent** - the load-bearing dispute; A did not revise |
| A3 | One repo need not sacrifice team ownership or independent release cadence. | CODEOWNERS; Bazel visibility; Nx independent releases | 0.91 → 0.84 | **conceded down** - A now concedes polyrepo has stronger isolation *by default*; monorepo autonomy requires deliberate project-scoped boundaries |
| A4 | Monorepo is a better onboarding/discoverability default (one tree, one graph, shared conventions). | Brito review; Google CACM | 0.87 → 0.82 | **conceded down** - a good software catalog (Backstage) narrows this; small residual edge in unified navigation/conventions |
| A5 | Monorepo converts hidden integration risk into visible **pre-merge** impact (affected-set schedules downstream checks in the same PR). | Nx affected; Bazel visibility | 0.90 | **converged** - both concede two different blast-radius profiles (see D4) |

## Seat B claims (champion: polyrepo)

| id | claim (compressed - full text in round0-seat-b-grok.md) | evidence | confidence | post-R1 status |
|----|-----|----|----|----|
| B1 | Polyrepo is the source-control encoding of DORA "loosely coupled teams" - the autonomy this org needs as it triples. | dora.dev; Accelerate/SODR 2021; Netflix; Amazon two-pizza | 0.86 | **surviving dissent** (feeds A2/D3) - A calls this "owner-unassigned weighting"; B holds criterion 2 dominates |
| B2 | A *working* monorepo here is not "Git + Nx" - it's an owned project graph + remote cache + merge-queue + a CI-owning team (Nx's seven questions), which the brief's "no bespoke build system / not Google-scale" constraint fights. | Google CACM (bill of materials); Merino; Nx KB; GHA required-checks failure mode | 0.90 → 0.85 | **surviving dissent** - the load-bearing dispute; B revised down but held |
| B3 | Versioned deps shrink blast radius: a bad change is an opt-in bump, not a HEAD-wide red `main`. | CACM; Nx/Graphite access tables; DORA cascading-failure | 0.84 → 0.82 | **converged** - B concedes A's pre-merge visibility; keeps post-merge/CI blast radius (see D4) |
| B4 | One repo on loosely related teams imposes a standing **consensus tax** (CI owner, version policy, git workflow, layout) that grows faster than the occasional cross-cutting PR. | Nx KB (seven org-wide questions); inverse-Conway; Amazon two-pizza | 0.81 (held) | **surviving dissent** (feeds A2/D3) |
| B5 | Atomic cross-project commits are a real loss but the *correct* loss given "occasional" cross-cutting; paying HEAD-coupling on every change to optimize the rare one is inverted ROI. | Netflix (semver + lockfiles + contract tests + mass-PR refactor) | 0.80 | **converged** with A1 - both agree the mechanism is real and frequency-dependent |
| B6 | Standardize polyrepo *now*: the 60-eng org is more team-shaped than product-shaped, and reversing a monorepo later is the expensive direction. | DORA inverse-Conway; CACM "unnecessary dependencies"; Merino | 0.78 | **minor surviving dissent** - A calls the asymmetry speculative; no data either way |

## Disputes and how Round 1 moved them

- **D1 - value of atomic cross-cutting change (A1 vs B5).** *Converged.* Both seats
  agree the atomic-commit mechanism is genuine and that for *loosely related*
  services it is the less-frequent case. Not decisive on its own; its weight is
  an empirical question about how often changes actually span projects here.
- **D2 - can this org run a monorepo without a bespoke build system / dedicated
  build team? (A2 vs B2).** **Unresolved - the core surviving dissent.** A holds
  it is attainable configuration (Nx affected + cache on GitHub Actions); B holds
  a *working* monorepo at this scale is an owned graph + remote cache +
  merge-queue + a CI-owning team the brief says the company may not staff. A did
  not revise A2; B revised B2's confidence down (0.90→0.85) but held the claim.
  Neither was overturned.
- **D3 - does team autonomy survive in a monorepo? (A3 vs B1/B4).** *Partly
  resolved by concession.* A conceded polyrepo has stronger isolation **by
  default** and revised A3 down; both agree monorepo autonomy is achievable only
  with deliberate project-scoped CI/ownership/deploy boundaries. The *magnitude*
  of the residual coordination ("consensus tax") is still disputed and rolls up
  into D2.
- **D4 - blast radius: pins vs HEAD-coupling (A5 vs B3).** *Converged on a
  tradeoff, not a winner.* A conceded pins contain immediate CI fallout and let
  consumers choose upgrade timing; B conceded the monorepo's pre-merge impact
  visibility. Result: **monorepo = impact visible before merge / one red `main`
  possible; polyrepo = breakage deferred to opt-in bump / version-drift risk.**
  Two different risk profiles, both legitimate.
- **D5 - onboarding/discoverability (A4 vs B3-catalog).** *Resolved by
  concession.* A software catalog (Backstage) largely neutralizes polyrepo's
  discovery cost; A revised A4 down. Small residual monorepo edge in unified
  navigation and shared conventions.
- **D6 - reversibility (B6 vs A's rebuttal).** *Minor surviving dissent.* Both
  directions can accrete expensive tangle (monorepo: unnecessary in-tree
  imports; polyrepo: version drift). No data; left open, low weight.

## Agreed by both seats (promoted to the verdict's common ground)

- Atomic cross-project commits are a **genuine** monorepo advantage - and are
  **less frequent** for loosely related services.
- Nx affected-graph + cache is a **real** mechanism, not vapor.
- Polyrepo has **stronger repository-level isolation by default**.
- If a monorepo is chosen, its **single biggest risk is the unified CI / `main`
  becoming an org-wide choke point**, and affected-only CI + caching + dependency
  boundaries + path ownership + independent deploy pipelines must be funded on
  **day one**, not deferred.
- If polyrepo is chosen, its **single biggest risk is shared-library version
  drift**, and an internal registry + Renovate/Dependabot + contract tests must
  be funded up front.

## Stop

Round 1 ran (seats were assigned opposing sides, so Round 0 could not agree).
After cross-examination, **the core dispute D2 survives** - neither seat
overturned the other on whether this specific org can operate a monorepo without
a bespoke build system or a dedicated build-platform team. The method does **not**
average the two verdicts. It records the surviving dissent and routes it to the
cheapest discriminating test - which, notably, **both seats independently
proposed in nearly identical form** (see [`verdict.md`](verdict.md)).
