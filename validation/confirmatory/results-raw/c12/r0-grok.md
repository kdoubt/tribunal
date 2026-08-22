**CLAIM:** Pre-PMF, the scarce resource is learning speed, not scale. Independent services tax that speed before the product shape is known.
**EVIDENCE:** An 8-person team already pays coordination cost. Microservices add deploy graphs, contracts, tracing, failure modes, and local-env sprawl. Those costs hit every pivot; they do not buy PMF.
**CONFIDENCE:** 0.9
**FALSIFIER:** First 8 weeks show >50% of cycle time blocked on *capacity/isolation* of a single, stable bounded context—not on product uncertainty or deploy plumbing.

**CLAIM:** Service cuts made before usage data freeze the wrong seams. A modular monolith lets you move those seams cheaply.
**EVIDENCE:** Pivots change who owns which data and which calls are sync. In one deployable, that is a module refactor. Across services it is API versioning, dual-writes, and distributed consistency.
**CONFIDENCE:** 0.85
**FALSIFIER:** Domain boundaries have been stable for ≥6 months *and* two groups already ship on independent cadences without stepping on the same data.

**CLAIM:** Eight engineers cannot staff N services as real products (on-call, CI, observability, data ownership) without starving the actual SaaS.
**EVIDENCE:** Each extra runtime is a product: pipeline, dashboards, runbooks. At this size you get a distributed monolith: coupling without isolation, plus more failure modes.
**CONFIDENCE:** 0.8
**FALSIFIER:** They already run ≥3 services in prod with independent deploys, clear SLOs, and feature throughput matching a single-app baseline.

**CLAIM:** A *modular* monolith is the option-preserving default: extract a service when a boundary is proven, not hoped.
**EVIDENCE:** Extraction is justified when one module has different scale/SLA/release needs *and* a stable interface. Until then, in-process modules give the same mental model without the network.
**CONFIDENCE:** 0.85
**FALSIFIER:** One module independently needs different scale or compliance *and* coupling to the rest is already the #1 cause of incidents or blocked releases.

**VERDICT: A** — Biggest risk: module boundaries rot into a ball of mud, so later extraction is a rewrite. Cheapest test you chose wrong: if, after ~3 months, a *single, named module* has a stable API *and* independently blocks the rest of the team on deploys/scale/incidents more than once a week, extract it; if that never happens, B would have been pure overhead.
