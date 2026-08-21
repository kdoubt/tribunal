# Round 0 - Seat A (Codex CLI, GPT-5-class) - champion: MONOREPO (A)

> Verbatim seat output. Isolated: this seat did not see Seat B's answer.
> **Role-incentive:** this seat was assigned to argue *for* the monorepo. Roles
> here are incentives, not personas - a real, different-vendor CLI citing real
> sources, told which side to push hardest. See [`README.md`](README.md).

### CLAIM 1

**CLAIM:** A monorepo is the strongest choice for cross-cutting changes because producers, consumers, tests, and migrations can change atomically.

**EVIDENCE:** A single commit can update a shared library and every affected service without publishing intermediate package versions or coordinating dependent PRs across repositories. A literature review identified simplified dependencies, cross-project coordination, and easier refactoring as principal monorepo benefits; Google describes its monorepo as a common source of truth. [Brito, Terra & Valente](https://arxiv.org/abs/1810.09477), [Google Research](https://research.google/pubs/why-google-stores-billions-of-lines-of-code-in-a-single-repository/)

**CONFIDENCE:** 0.96

**FALSIFIER:** Cross-project changes remain extremely rare, and shared libraries expose stable contracts that almost never require coordinated consumer updates.

### CLAIM 2

**CLAIM:** At 60 engineers and roughly eight-plus projects, monorepo CI is an attainable configuration problem, not a bespoke-build-system problem.

**EVIDENCE:** Off-the-shelf tooling already computes the affected project set from Git changes and a dependency graph, then runs only relevant tasks; caching further reduces repeated work. The important mechanism is dependency-aware selection rather than rebuilding the repository wholesale. Nx explicitly warns that high-fan-out library changes can still affect nearly everything, making dependency discipline and caching mandatory. [Nx affected documentation](https://nx.dev/docs/features/ci-features/affected)

**CONFIDENCE:** 0.88

**FALSIFIER:** The company's languages or build systems cannot be represented reliably in a supported project graph, causing frequent false negatives or routine full-repository rebuilds.

### CLAIM 3

**CLAIM:** One repository need not sacrifice team ownership or independent release cadence.

**EVIDENCE:** Directory-level `CODEOWNERS` rules can automatically request—and branch protection can require—approval from the responsible team. Build-level visibility can make undeclared cross-team dependencies fail during analysis. Separately, existing monorepo tooling supports project-specific versions, tags, changelogs, and release selection, so repository atomicity does not imply synchronized deployment. [GitHub CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners), [Bazel visibility](https://bazel.build/concepts/visibility), [Nx independent releases](https://nx.dev/docs/guides/nx-release/release-projects-independently)

**CONFIDENCE:** 0.91

**FALSIFIER:** Teams require repository-level security isolation rather than review and dependency boundaries, or the release platform cannot deploy artifacts independently from a shared commit history.

### CLAIM 4

**CLAIM:** A monorepo gives this rapidly hiring organization a better onboarding and discoverability default.

**EVIDENCE:** New engineers receive one searchable source tree, one dependency graph, and shared build/test conventions instead of learning repository discovery, version-publication, and CI conventions repeatedly. The literature review identifies centralization, standardization, simplified organization, and contributor-wide visibility as defining monorepo characteristics; Google identifies the repository as a common source of truth. [Monorepo literature review](https://arxiv.org/abs/1810.09477), [Google Research](https://research.google/pubs/why-google-stores-billions-of-lines-of-code-in-a-single-repository/)

**CONFIDENCE:** 0.87

**FALSIFIER:** The organization fails to enforce repository-wide conventions, leaving each directory with unrelated commands and undocumented ownership—one repository physically, but many repositories operationally.

### CLAIM 5

**CLAIM:** For shared-code evolution, a monorepo converts hidden integration risk into visible pre-merge impact.

**EVIDENCE:** The project graph identifies downstream consumers of a changed library and schedules their checks in the same PR. In polyrepo, equivalent assurance requires publishing or constructing a candidate dependency, locating consumers, and coordinating their pipelines. Nx documents the graph traversal mechanism; Bazel documents analysis-time rejection of forbidden dependencies. [Nx affected documentation](https://nx.dev/docs/features/ci-features/affected), [Bazel visibility](https://bazel.build/concepts/visibility)

**CONFIDENCE:** 0.90

**FALSIFIER:** A measured pilot shows affected-set computation is unreliable, while the existing polyrepo dependency-update automation consistently tests every downstream consumer before merge with less operational effort.

### VERDICT INPUT

**A — MONOREPO.**

**Single biggest risk:** The unified CI and main branch become an organization-wide choke point. A high-fan-out shared-library change, incorrect dependency graph, or flaky global check can block every team. The company must therefore establish affected-only CI, caching, explicit dependency boundaries, path ownership, and independent deployment pipelines as foundational infrastructure—not deferred optimization.
