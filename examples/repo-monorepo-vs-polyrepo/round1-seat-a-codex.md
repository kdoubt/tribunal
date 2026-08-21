# Round 1 - Seat A (Codex CLI) - cross-examination of the polyrepo case

> Verbatim seat output. Seat A received Seat B's Round 0 claims (relayed as "the
> opposing seat's claims") plus its own Round 0 as stateless context, and was
> asked to ATTACK / CONCEDE / REVISE / give a VERDICT INPUT. Still championing A.

1. ATTACK

Claim 1 conflates repository boundaries with deployability. DORA prescribes loose architectural and team coupling, not polyrepo; versioned artifacts and independent pipelines work inside a monorepo. Conversely, separate repositories do not prevent release coordination when APIs or behavior are coupled. Netflix demonstrates one successful organizational design at vastly different scale, not that polyrepo causes autonomy. "Criterion 2 therefore dominates" is owner-unassigned weighting, not evidence.

Claim 2 attacks Google-scale monorepos rather than this 60-engineer case. Eight-plus projects do not require Piper, CitC, Bazel, or Google's traffic profile. Nx/GitHub Actions can calculate affected projects and cache tasks without a bespoke build system. The path-filtered-required-check problem has standard workflow-level workarounds and is not intrinsic to monorepos. Meanwhile, polyrepo CI is not merely commodity: shared templates, registry operation, dependency automation, contract testing, cross-repository credentials, and fleet-wide policy rollout form their own platform. Its falsifier is over-specific and stacked to be nearly unfalsifiable.

Claim 3 overstates blast-radius reduction. Pins delay propagation, but they also hide integration failures until consumer upgrade time; this converts immediate visible breakage into distributed version drift. A monorepo change need not deploy every consumer, and pre-merge downstream checks prevent a bad shared-library commit from reaching `main`. Repository permissions are genuinely stronger security isolation, but "one merge queue" and "one broken workflow" are design choices, not necessary properties. A catalog restores lookup, not atomic navigation, consistent conventions, or reliable dependency discovery.

Claim 4 mistakes shared governance for unique monorepo overhead. Polyrepo still needs organization-wide decisions about templates, dependency policy, supported toolchains, security controls, ownership, and releases—then must propagate them across repositories. "One pipeline" need not mean one serialized workflow: project-scoped jobs and independent deployments preserve operational autonomy. The inverse-Conway argument supplies no mechanism showing that a shared Git boundary creates runtime coupling.

Claim 5 correctly identifies the tradeoff but does not establish its ROI. "Occasional" cross-cutting changes may still be expensive or risky, especially across shared libraries; no frequency, duration, or incident data supports "cheaper." Semver, contract tests, publisher feedback, registries, and mass PRs are substantial machinery and cannot make a logically atomic multi-repository change.

Claim 6's migration asymmetry is speculative. Polyrepo can accumulate undocumented dependency drift, duplicated tooling, and incompatible conventions that are also costly to consolidate. Monorepos can enforce visibility boundaries that prevent "free" imports. Git submodules do not recreate atomic commits or a unified dependency graph. Headcount growth alone does not imply repository proliferation.

2. CONCEDE

Polyrepo provides stronger repository-level access isolation, smaller checkout scope, and autonomy by default. Version pins contain immediate CI fallout and let consumers choose upgrade timing. Because these services are loosely related, monorepo's atomic-change advantage will be less frequent than in a tightly integrated product. The monorepo's largest genuine risk remains shared CI becoming a flaky, slow organizational choke point; explicit ownership, dependency boundaries, affected-only CI, and independent deployment are mandatory.

3. REVISE

Claim 3 — New text: "A monorepo can preserve team ownership and independent releases, but requires deliberate project-scoped CI, deployment, ownership, and dependency boundaries; polyrepo provides stronger isolation by default." Confidence 0.91 → 0.84.

Claim 4 — Confidence 0.87 → 0.82: a good service catalog narrows monorepo's discoverability advantage, though it does not reproduce unified navigation and conventions.

4. VERDICT INPUT

A — cheapest settling test: run a two-week monorepo pilot containing one shared library and two representative services, measuring affected-CI time, unrelated failures, and effort for one cross-cutting change against the current polyrepo workflow.
