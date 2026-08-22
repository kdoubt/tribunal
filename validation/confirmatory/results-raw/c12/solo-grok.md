DECISION MEMO — C12: Initial architecture for a new SaaS product

CALL
A

An 8-engineer, pre-PMF team should ship a modular monolith: one deployable, one datastore, explicit module boundaries (packages/domains, not network hops). Independent deploys, service mesh, and per-service infra are premature until the product and ownership map stop moving.

Tradeoffs
A keeps change cheap: one deploy, one set of logs/metrics, local transactions, and refactor-by-move-folder when a pivot redraws the domain. The cost is a growing blast radius and the temptation to let modules leak. B buys independent scaling and team isolation, but prices every feature as a distributed system: contracts, auth, tracing, failure modes, and CI/CD multiplied by service count. At 8 people and unknown product shape, that tax is paid in iteration speed, not in reliability.

Biggest risk
The monolith becomes a ball of mud: module boundaries rot, so a later split is a rewrite instead of an extraction.

Cheapest wrong-choice test
If, for two consecutive quarters after PMF, two distinct domains must ship on independent cadences (different SLOs, different on-call, different scale curves) and a module change regularly blocks the other domain’s release, extract those two services. Until that coupling is measured and painful, stay on A. If you cannot name those two domains today, B was already the wrong bet.
