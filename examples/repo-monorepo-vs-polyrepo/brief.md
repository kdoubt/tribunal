# Frozen Brief - repository strategy for a scaling org: monorepo vs polyrepo

## Question under review

A B2B software company, ~25 engineers today across 8 loosely related services/apps (some shared libraries), growing to ~60 engineers in a year across more teams. They must standardize NOW on one source-control strategy for all product code:

- **A) Monorepo** - all services, apps, and shared libraries in ONE repository, with unified tooling, one CI graph, and atomic cross-project commits.
- **B) Polyrepo** - one repository per service/library/team, versioned dependencies between them, independent CI and release per repo.

## Decision criteria (owner-supplied - genuinely balanced; there is NO industry consensus here)

1. Ease of cross-cutting changes and code sharing / refactoring across projects.
2. Team autonomy and independent release cadence as teams multiply.
3. Tooling and CI complexity/cost - what the org must build and operate to make the choice work at 60 engineers.
4. Onboarding, discoverability, and "blast radius" of a bad change.

Weigh these against each other. This is a real judgment call where excellent engineers disagree - **commit to A or B**, do not retreat to "it depends" or "do both". Name the single biggest risk of your choice.

## Constraints

- One strategy for all product code (no per-team free choice).
- The company will invest in *some* tooling either way, but is not Google/Meta-scale and cannot build a bespoke build system.
- Services are loosely related (some shared libs, occasional cross-cutting changes), not a tightly-coupled single product.

## Output contract for seats

Max 6 claims, each: CLAIM / EVIDENCE (concrete mechanism, named practice/source as EXTERNAL, or specific failure mode) / CONFIDENCE (0-1) / FALSIFIER. Plus VERDICT INPUT: **A or B** + single biggest risk.

## Role assignment (this panel uses role-incentives to stress-test both sides)

This is a genuinely-contested, no-consensus decision, so the orchestrator has assigned opposing **role-incentives** (roles as incentives, not personas - both seats are real, different-vendor CLIs citing real evidence; the roles only decide which side each argues hardest):
- **Seat A: champion option A (monorepo).** Make the strongest evidence-grounded case FOR the monorepo.
- **Seat B: champion option B (polyrepo).** Make the strongest evidence-grounded case FOR the polyrepo.

Argue your assigned side hard, with real mechanisms and cited sources. You MAY concede a genuine tradeoff, but do not hedge into "it depends" and do not fabricate evidence. Your VERDICT INPUT is still A or B (it will be your assigned side unless the evidence genuinely forces otherwise - say so if it does).
