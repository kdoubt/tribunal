# Round 1 - Seat B (Grok CLI) - cross-examination of the monorepo case

> Verbatim seat output, except one leading tool-narration line (the seat
> narrating that it would read the brief and both Round 0 files) was trimmed
> from the top; the argument is untouched. Seat B received Seat A's Round 0
> claims plus its own Round 0 as stateless context, and was asked to ATTACK /
> CONCEDE / REVISE / give a VERDICT INPUT. Still championing B.

**1. ATTACK**

A's Claim 1 (atomic commits, conf 0.96) is a real mechanism aimed at the wrong frequency. The brief already says loosely related services and *occasional* cross-cutting; A's own falsifier ("cross-project changes remain extremely rare") is approximately that brief. 0.96 treats criterion 1 as if it dominates. Google's "common source of truth" is Piper/CitC/a test farm, not a pointer to this shop. Brito et al. list *reported* monorepo benefits; they are not a cost model for 25→60 with no bespoke build system.

A's Claim 2 is the load-bearing over-argument. "Nx affected + cache = configuration, not a platform" has no pointer to a 50–70 engineer, 8+ loosely related *polyglot* estate running that on unmodified GitHub Actions with no dedicated owner. Nx's own monorepo-vs-polyrepo note still asks for a dedicated team and seven org-wide answers, or split. Path-filtered jobs vs required checks is a known GitHub failure mode. Confidence 0.88 sits on a falsifier A underweights: eight loosely related services will not all sit in one supported project graph.

A's Claim 3 contradicts Claim 2. CODEOWNERS is review routing, not DORA "deploy independently of services you depend on." HEAD-coupled shared libraries fail that test by design. Bazel *visibility* is then cited as the enforcement mechanism — that is the bespoke build system the brief forbids. Nx independent releases can tag artifacts; they do not uncouple one `main`, one merge queue, one root workflow. "Need not sacrifice" is a policy hope. 0.91 is too high for a can-in-theory claim with no production pointer at this constraint set.

A's Claim 4 (onboarding via one tree, 0.87) has no mechanism after controlling for a catalog. Discoverability is Backstage's job; cloning eight other teams' graphs, linters, and flakes is cognitive blast radius. Google again is Code Search, not `git clone`. A's falsifier — fail to enforce conventions and you have many repos operationally — is the standing consensus tax, not a side note.

A's Claim 5 (pre-merge affected-set, 0.90) assumes the graph Claim 2 has not shown this org can keep. Nx already warns high-fan-out library changes still hit nearly everything — which is the shared-lib case. The polyrepo alternative is not A's strawman of hand-built candidate packages; it is lockfiles, contract tests, and Renovate. Visibility of impact is not the same as forcing every consumer onto the producer's clock.

A names the unified CI/`main` choke point as the biggest risk and then still picks A. That risk is the argument *against* A under "cannot build a bespoke build system."

**2. CONCEDE**

Atomic cross-project commits are a genuine monorepo advantage (criterion 1). Nx affected-graph + cache is a real mechanism, not vapor. CODEOWNERS can route reviews. Project-level tags can deploy without a single release train. Pre-merge consumer checks in one PR, *if the graph is honest*, beat surprise breakage at pin-bump time. A's named choke-point risk should survive into the final answer as the thing a monorepo must fund on day one.

**3. REVISE**

Claim 2 — new text: A working monorepo here is not Piper, but it is also not "turn on Nx." It is a owned project graph, remote cache, merge-queue policy, and a team that answers Nx's seven questions, on a polyglot estate GitHub Actions does not represent for free. Polyrepo CI remains the commodity path (per-repo GHA, registries, Renovate). Confidence 0.90 → 0.85.

Claim 3 — add: concede A's pre-merge visibility; keep post-merge/CI blast radius (one red `main`/workflow vs an opt-in bump). Confidence 0.84 → 0.82.

Claims 1, 4, 5, 6: no revision. Criterion 2 still dominates given "loosely related" and "more teams."

**4. VERDICT INPUT**

B. Cheapest test: two-week spike of two services + one shared lib in one GitHub repo with Nx affected and CODEOWNERS; if median PR CI stays within ~20% of today's per-repo CI and unrelated-red hours stay near zero with no dedicated owner, A's Claim 2 stands.
