DECISION MEMO — C19

CALL

A

Standardize on Terraform.

Terraform best matches the organization’s current center of gravity: ops-leaning engineers, polyglot application teams, and a need for one broadly understood infrastructure interface. HCL constrains implementation choices, keeps reviews focused on infrastructure intent, and avoids making language ecosystems, package managers, and software architecture patterns part of the platform contract. Its mature provider ecosystem, hiring pool, documentation, and operational tooling reduce adoption and integration risk.

Key tradeoffs and risks:

- Complex abstractions are less elegant than in Pulumi/CDK; modules and `for_each` can become awkward or opaque.
- HCL adds a domain-specific language that application developers must learn.
- Terraform state, provider upgrades, and module-version governance require disciplined platform ownership.
- General-purpose IaC offers better testing, refactoring, IDE support, and reuse—but also permits arbitrary complexity, framework-building, dependency drift, and inconsistent language choices across teams.
- CDK-style tools can improve developer velocity inside one cloud ecosystem, but weaken portability and create stronger coupling to that ecosystem.

Single biggest risk:

Terraform’s constrained language causes teams with dynamic or highly repetitive infrastructure to build brittle module hierarchies, generate HCL, or bypass the standard entirely. The failure mode is not verbosity; it is loss of adoption because the approved path becomes materially harder than writing infrastructure in normal code.

Cheapest test / wrong-choice criterion:

Run a two-week pilot with one representative app team building a moderately dynamic service stack using Terraform: multiple environments, conditional resources, repeated components, and policy checks. Choose B instead if the team must generate HCL or introduce more than one bespoke wrapper/tool to keep the configuration maintainable, or if independently measured implementation time exceeds the equivalent Pulumi/CDK prototype by more than 30% after review and operational requirements are held constant.
