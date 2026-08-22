CLAIM: A modular monolith best supports rapid iteration before product-market fit.  
EVIDENCE: One deployable keeps feature work, refactoring, debugging, and transactional changes straightforward while requirements and domain boundaries remain unstable.  
CONFIDENCE: High.  
FALSIFIER: Most product changes already belong to stable, independently evolving domains and rarely require cross-domain coordination.

CLAIM: Microservices would impose disproportionate overhead on an eight-engineer team.  
EVIDENCE: Service ownership, deployment pipelines, observability, networking, failure handling, contract versioning, and local development consume capacity regardless of customer value.  
CONFIDENCE: High.  
FALSIFIER: A mature internal platform makes service creation and operation nearly free, and the team already has strong distributed-systems expertise.

CLAIM: Clear internal module boundaries preserve a practical migration path.  
EVIDENCE: Enforced dependency rules and explicit module APIs reduce coupling, allowing a genuinely constrained component to be extracted later without paying distributed-systems costs everywhere today.  
CONFIDENCE: Medium-high.  
FALSIFIER: Modules cannot be independently tested or changed because business workflows and data access remain pervasively entangled.

CLAIM: Independent deployment is valuable only when it solves a demonstrated bottleneck.  
EVIDENCE: At this scale, coordination is more likely driven by uncertain product requirements than by deployment contention, heterogeneous scaling, or fault-isolation needs.  
CONFIDENCE: High.  
FALSIFIER: A specific subsystem already requires materially different scaling, security isolation, availability, technology, or release cadence.

VERDICT: A — Biggest risk: the “modular” monolith degrades into a tightly coupled big ball of mud. Cheapest discriminating test: track cross-module coupling for several sprints; if routine changes repeatedly require coordinated edits across unrelated modules—or one module demonstrably needs independent scaling, isolation, or deployment—extract that module and reconsider the architecture.
