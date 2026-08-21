# Round 0 - Seat B (Grok CLI, Grok-4-class) - champion: POLYREPO (B)

> Verbatim seat output, except one leading tool-narration line (the seat
> narrating its own source-gathering) was trimmed from the top; the argument is
> untouched. Isolated: this seat did not see Seat A's answer.
> **Role-incentive:** this seat was assigned to argue *for* the polyrepo. Roles
> here are incentives, not personas - a real, different-vendor CLI citing real
> sources, told which side to push hardest. See [`README.md`](README.md).

# Seat B — Champion B (polyrepo)

Round 0 independent. Assigned side is B; the evidence matches the brief (loosely related services, occasional cross-cutting, no bespoke build system, 25→60 engineers). I am not retreating to hybrid.

---

### CLAIM 1
**Polyrepo is the source-control encoding of independently deployable teams — the DORA capability this org is about to need as it triples headcount.**

**EVIDENCE.** DORA's core "loosely coupled teams" capability is a set of *operational* tests, not a repo aesthetic: a team can change its system without outside permission; complete work without fine-grained coordination; **deploy and release on demand, independently of services it depends on and of services that depend on it**; test on demand without a shared integration environment ([dora.dev/capabilities/loosely-coupled-teams](https://dora.dev/capabilities/loosely-coupled-teams/)). The 2021 Accelerate State of DevOps report called loosely coupled architecture "one of the strongest predictors of successful continuous delivery," with elite performers meeting reliability targets **three times more likely** to use it than low performers ([cloud.google.com/resources/state-of-devops](https://cloud.google.com/resources/state-of-devops)).

The mechanism: **versioned artifacts + per-repo CI** make "independent of the services it depends on" the default. A consumer ships against last week's library tag; a producer ships today's binary. A monorepo with one CI graph and HEAD-coupled shared libraries inverts that: every library change is immediately every consumer's problem, which is exactly DORA's tightly-coupled failure mode ("small changes can result in large-scale, cascading failures" requiring constant coordination). Netflix stated the same mechanism in production: "Our microservice architecture decouples engineering teams from each other, allowing them to build, test and deploy their services as often as they want" — and they kept **distributed repositories** rather than collapsing into a monorepo, because a monorepo "would challenge our culture of Freedom and Responsibility" and impose "gates to deploying changes" ([Netflix TechBlog, 2017](https://netflixtechblog.com/towards-true-continuous-integration-distributed-repositories-and-dependencies-2a2e3108c051)). Graphite's own decision guide maps this brief almost verbatim: use polyrepo for "autonomous teams working on loosely coupled services" and "independent lifecycles" ([graphite.com/guides/monorepo-vs-polyrepo-pros-cons-tools](https://graphite.com/guides/monorepo-vs-polyrepo-pros-cons-tools)). Amazon's two-pizza model is the org-chart version of the same cut: single-threaded ownership of one service, "from ideation to execution" ([aws.amazon.com/executive-insights](https://aws.amazon.com/executive-insights/content/amazon-two-pizza-team/)).

This brief's services are "loosely related," not one product. Criterion 2 (autonomy and independent cadence as teams multiply) therefore dominates criterion 1.

**CONFIDENCE:** 0.86  
**FALSIFIER:** Measured DORA metrics (lead time, deploy frequency, change-fail rate) at this company over 12 months after a monorepo adoption beat a polyrepo baseline *without* a dedicated build-platform team, showing HEAD-coupling did not become a coordination bottleneck as teams multiplied.

---

### CLAIM 2
**A working monorepo under this constraint set is not "Git plus Nx." It is a platform product Google had to invent. Polyrepo CI is commodity; monorepo-at-HEAD CI is not.**

**EVIDENCE.** Google's CACM paper is the canonical monorepo success story, and it is also the bill of materials: they could not find a commercial or open-source VCS that held the model, so they built **Piper** (on Bigtable, then Spanner, Paxos-replicated across data centers), **CitC** (cloud FUSE workspace), trunk-at-HEAD, and a distributed build-and-test farm serving ~500k file-read QPS on a workday. Drawbacks they list first: "having to create and scale tools for development and execution and maintain code health" (Potvin & Levenberg, CACM, [cacm.acm.org](https://cacm.acm.org/research/why-google-stores-billions-of-lines-of-code-in-a-single-repository/)). Julio Merino, writing from inside that world: "monorepos require dedicated teams (plural) *and* tools to run nicely (not just random engineers 'volunteering' their efforts), and these cost a lot of time and money… If you cannot afford the cost, it might be better to stay with multiple smaller repositories" ([jmmv.dev, 2021](https://jmmv.dev/2021/02/google-monorepos-and-caching.html)). The mechanism he names: a monorepo CI that "blindly do[es] builds from the root" inflates build times until the process stalls; the polyrepo fix is **binary package versions as synchronization points** — you reuse *other teams' already-done builds*. Google recreates those sync points with a cross-user remote action cache and hermetic Bazel; that is the bespoke build system this brief says the company **cannot** build.

The 25→60 shop therefore faces a fork: (a) naive monorepo CI that rebuilds too much, fights GitHub Actions required-checks vs path-filters (a well-known GHA failure mode: path-filtered jobs skip and **block** required status checks — [github.com/orgs/community/discussions/44490](https://github.com/orgs/community/discussions/44490)), or (b) invest in affected-graph tooling, remote cache, merge queues, and a team to own the root pipeline. Nx, a monorepo vendor, still says a single org-wide monorepo needs "a dedicated team" for shared CI and libraries once you are at hundreds of developers — and lists seven organizational questions (dependency policy, CODEOWNERS, module boundaries, folder structure, git workflow, who owns CI, deploy coupling) that **must** be answered in consensus or the repo should be split ([nx.dev/docs/kb/monorepo-vs-polyrepo](https://nx.dev/docs/kb/monorepo-vs-polyrepo)).

Polyrepo's tooling stack at this scale is already on the shelf: GitHub/GitLab per-repo CI, npm/Maven/PyPI/Cargo/NuGet, Docker registries, Dependabot/Renovate. That is what "the company will invest in *some* tooling" should buy — package publishing and a catalog — not a Piper-lite.

**CONFIDENCE:** 0.90  
**FALSIFIER:** A 50–70 engineer org with 8+ loosely related polyglot services runs a single Git monorepo on unmodified GitHub Actions (no Bazel/Pants, no remote cache farm, no dedicated build-platform team) with median PR CI under ~15 minutes and no org-wide red-main incidents attributable to unrelated projects, sustained for a year.

---

### CLAIM 3
**Versioned dependencies shrink blast radius: a bad change becomes an opt-in bump, not a HEAD-wide outage of everyone else's CI and main.**

**EVIDENCE.** Mechanism: producer publishes `lib@1.4.2`; consumers pin via lockfile; they absorb the change on *their* release train. A broken 1.4.2 does not turn eight other pipelines red. A monorepo-at-HEAD shared library does the opposite by design: "Immediately after any commit, the new code is visible to, and usable by, all other developers" (Potvin & Levenberg, CACM). That is a feature for atomic refactors and a defect for blast radius.

Nx's comparison table is explicit: monorepo access is "everyone sees all code"; polyrepo is "repository-level permissions per team." Graphite lists the same: "granular access control… enforced at the repo level," plus a smaller Git footprint (clone/fetch/diff of one service, not the union). DORA again: in a tightly coupled architecture, "small changes can result in large-scale, cascading failures." Per-repo branch protection, required reviewers, and GitHub org secrets are the commodity controls; folder-level CODEOWNERS in a shared repo are a weaker substitute that still leaves one merge queue, one `main`, and one broken workflow file in the way of every team.

Onboarding blast radius is the same mechanism in cognitive form: a new hire clones *their* service, runs *its* CI, ships. They do not inherit eight other teams' build graph, lint stack, and flaky tests. Discoverability of "what else exists" is a **catalog** problem; Spotify built Backstage for exactly that in a many-repo estate — "Makes all the software in your company, and who owns it, discoverable" ([backstage.io](https://backstage.io/docs/features/software-catalog/)) — without forcing those services to share a Git tree.

**CONFIDENCE:** 0.84  
**FALSIFIER:** Incident review shows a higher rate of customer-facing regressions from *stale pinned* library versions in polyrepo than from HEAD-coupled library changes in a comparable monorepo, *and* new-hire time-to-first-PR is worse in polyrepo after controlling for a software catalog.

---

### CLAIM 4
**Forcing one repo on loosely related teams imposes a standing consensus tax (CI owner, version policy, git workflow, folder layout) that grows faster than the occasional cross-cutting PR.**

**EVIDENCE.** Nx — again, a monorepo vendor — states the decision is "mostly organizational," then lists the questions that **must** be agreed or "they'll need to work in separate repositories": single-version vs independent versions; who reviews what; which projects may depend on which; folder naming; trunk vs long-lived branches; **who maintains the CI pipeline**; independent vs coupled deploys ([nx.dev](https://nx.dev/docs/kb/monorepo-vs-polyrepo)). Those are not one-time RFC items. At 25 people they are annoying; at 60 people across more teams they are a standing politics surface: every toolchain bump, every Actions runner change, every "please don't use that linter plugin" fight lands in one PR queue.

The brief's services are *loosely related*. That is Nx's polyrepo trigger: "teams share little code and nothing should force them to move together." A shared `main` plus a unified CI graph *does* force them to move together at the source-control layer even if production deploys are tagged independently. Merge-queue capacity and unrelated-test flakes become a commons. That is the inverse-Conway failure: the communication structure (one repo, one pipeline, one set of conventions) will copy itself into coupling the services did not need.

Amazon's two-pizza design exists to *prevent* that commons: ownership of one service, not "complex systems or solving problems spanning multiple services" ([AWS Executive Insights](https://aws.amazon.com/executive-insights/content/amazon-two-pizza-team/)). Polyrepo makes that ownership inspectable (repo permissions, CODEOWNERS of one tree, one release dashboard). Monorepo ownership is a policy file hoping people behave.

**CONFIDENCE:** 0.81  
**FALSIFIER:** After 18 months in one monorepo, teams report *lower* coordination cost on tooling/CI/conventions than a matched polyrepo org, with a single pipeline-owning team of ≤2 people keeping median wait-on-unrelated-CI near zero.

---

### CLAIM 5
**Criterion 1 is a real loss — atomic cross-project commits — and it is the correct loss given "occasional" cross-cutting and "loosely related." Paying HEAD-coupling on every change to optimize the rare one is inverted ROI.**

**EVIDENCE.** Conceded mechanism: a monorepo can land API + all in-tree consumers in one commit; polyrepo needs N PRs, a published version, and consumer bumps. Graphite and Nx both name this as the monorepo's main technical advantage. Netflix named the same pain — "managing dependencies is hard" (breaking APIs, transitive JVM classpath collisions, behavioral breaks) — and **still rejected** both "share little" *and* monorepo. Their alternative, which is the polyrepo playbook at this scale: **semver + dependency locking** ("should be sufficient for most teams to prevent [breaking API changes], assuming cultural rigor around semver"), publisher-feedback / consumer contract tests, and "distributed refactoring" via mass PRs across Git repos ([Netflix TechBlog, 2017](https://netflixtechblog.com/towards-true-continuous-integration-distributed-repositories-and-dependencies-2a2e3108c051)).

The brief is not Google's tightly-shared C++/Java corpus. It is "some shared libs, occasional cross-cutting." Treat shared libraries as *products with versions*, not as folders. The investment the company *will* make should be: an internal registry, Renovate/Dependabot across the org, contract tests, and a thin template (cookiecutter / Nx plugin / copier) so CI config is not copy-pasted by hand. That is cheaper than a correct affected-graph CI, and it keeps the default path — independent release — aligned with how the services actually relate.

If cross-cutting were the *dominant* weekly activity, this claim would flip. The brief says it is not.

**CONFIDENCE:** 0.80  
**FALSIFIER:** Time-and-motion on a representative quarter shows engineers spending more hours on cross-repo version coordination than they would have spent on monorepo merge-queue / affected-CI / convention fights, *and* a majority of those hours are on changes that would have been a single atomic commit.

---

### CLAIM 6
**Standardize on polyrepo *now* because the 60-engineer org will be more team-shaped, not more product-shaped; reversing a monorepo later is the expensive direction.**

**EVIDENCE.** Headcount 25→60 with "more teams" is Conway pressure toward *more* service/team boundaries, not toward one product. DORA's inverse-Conway maneuver: design team structure to produce the architecture you want — here, loosely coupled services ([dora.dev](https://dora.dev/capabilities/loosely-coupled-teams/)). Putting all product code in one Git repo now teaches every new team that the unit of review, CI, and convention is the whole company. Splitting a grown monorepo later means cutting a tangled import graph that grew *because* in-tree imports were free (the CACM paper lists "unnecessary dependencies" / codebase complexity as a primary monorepo drawback; Merino: "untangling the dependency mess that arises in such an environment can be next to impossible").

Polyrepo's known failure mode is the opposite tangle — version drift — and it is **reversible incrementally**: extract a shared lib, publish, tighten semver, add Renovate. You can also later *compose* a workspace (git submodules, package-based tools, or an IDP catalog) without a flag-day migration of history. The constraint "one strategy for all product code" is satisfied by **one policy**: one repo per deployable/library, versioned deps, org-standard CI template, catalog for discovery. That is a strategy, not per-team anarchy.

**CONFIDENCE:** 0.78  
**FALSIFIER:** Within a year the 8 services have collapsed into one tightly coupled product with daily cross-cutting refactors as the modal engineering activity; under that new shape, a monorepo's atomic-commit advantage would dominate.

---

## VERDICT INPUT

**B (polyrepo)**

**Single biggest risk:** **version drift on shared libraries** — diamond dependencies, delayed security/bugfix adoption, and "works in my repo" against an old pin. Netflix documented this as the hard problem they chose to *tool* (semver, lockfiles, publisher feedback, mass-PR refactors) rather than dissolve with a monorepo. If the company under-invests in registry + automated upgrades + contract tests, polyrepo's autonomy becomes an unpatchable estate. That is the risk to fund, not a reason to buy a Piper-shaped CI graph this company is constrained not to build.
