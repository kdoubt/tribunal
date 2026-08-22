CLAIM: Terraform is the safer organization-wide default for an ops-leaning, polyglot team.  
EVIDENCE: HCL is language-neutral and deliberately constrained, producing more uniform, reviewable infrastructure definitions than choosing among TypeScript, Python, Go, or C#. It avoids importing each application team’s language ecosystem and coding conventions into platform operations.  
CONFIDENCE: High  
FALSIFIER: Most infrastructure contributors are already proficient in one shared general-purpose language and consistently find HCL—not cloud complexity—to be the delivery bottleneck.

CLAIM: Terraform’s ecosystem and operating model reduce adoption and integration risk.  
EVIDENCE: Its broad provider/module ecosystem, established state and plan workflow, extensive documentation, and widespread practitioner familiarity make uncommon services and hiring easier to support.  
CONFIDENCE: High  
FALSIFIER: A representative inventory shows material provider gaps or poor provider quality for the organization’s critical infrastructure, while Pulumi/CDK supports it reliably.

CLAIM: “Real code” improves abstraction power but increases governance burden.  
EVIDENCE: General-purpose languages enable strong testing, reusable libraries, IDE support, and sophisticated composition. They also make hidden control flow, dependency sprawl, framework construction, and over-abstraction easier—costs that are especially relevant for ops-leaning reviewers.  
CONFIDENCE: Medium-high  
FALSIFIER: A trial demonstrates substantially lower duplication and lead time without reducing plan readability, review quality, or incident diagnosability.

CLAIM: Terraform’s main weakness is abstraction friction at scale.  
EVIDENCE: Complex transformations, reusable platform products, and application-integrated infrastructure can become awkward in HCL, encouraging copy-paste or brittle module interfaces.  
CONFIDENCE: High  
FALSIFIER: Teams can express representative complex stacks with shallow modules, little duplication, and no external code generation.

VERDICT: A — Terraform. Biggest risk: HCL/module constraints become a productivity ceiling as app developers demand richer abstractions. Cheapest discriminating test: have both approaches implement and modify the same representative stack—including loops, conditional resources, reusable components, policy checks, and import of an existing resource—then compare reviewer comprehension and change lead time. Choose B instead if it is materially faster while its preview remains equally understandable to ops reviewers.
