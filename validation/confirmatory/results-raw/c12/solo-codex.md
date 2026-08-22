DECISION MEMO — C12: Initial SaaS Architecture

CALL

A

Build a modular monolith: one deployable application with explicit module ownership, narrow internal interfaces, and isolated persistence access where practical.

The dominant pre-PMF constraint is learning speed, not independent scaling. For eight engineers, microservices would impose disproportionate costs: service contracts, distributed tracing, deployment orchestration, local-development complexity, network failure handling, data consistency, and cross-service migrations. Those costs slow pivots precisely when domain boundaries are least understood.

A modular monolith keeps transactions, debugging, testing, and deployment simple while still forcing architectural discipline. It also avoids prematurely encoding speculative product boundaries into network APIs. If modules have clear interfaces and do not reach into each other’s data, high-pressure components can later be extracted with manageable effort.

Key tradeoffs and risks:

- Deployments are coupled; a small change redeploys the whole application.
- A defect or resource spike can affect the entire product.
- Independent scaling and technology choices are limited.
- Weak enforcement can turn “modular” into a tightly coupled big ball of mud.
- Later extraction has a real migration cost—but likely less than repeatedly reshaping premature services during discovery.

Single biggest risk:

The monolith’s module boundaries erode under iteration pressure, creating tangled data ownership and dependencies that make changes unsafe and service extraction expensive.

Cheapest test that says the choice was wrong:

Track deployment interference for four consecutive weeks. If two or more teams repeatedly need independent release timing or scaling, and the shared deployable causes at least one blocked release, rollback, or production incident per week despite clear module boundaries, extract the specific offending module. That evidence—not projected scale—means the monolith has become an active delivery constraint.
