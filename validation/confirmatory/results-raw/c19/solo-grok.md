DECISION MEMO — C19: Terraform vs general-purpose IaC

CALL
A

Standardize on Terraform (HCL). Keep CDKTF/Pulumi off the critical path. Ops-majority orgs pay a tax on every plan, review, hire, and incident if the IaC language is a general-purpose one. HCL is the shared dialect; the provider catalog, state/plan contract, and hiring market are the product. App-dev involvement is real, but they should consume versioned modules and CI-gated PRs, not author TypeScript that emits clouds.

Tradeoffs
- Terraform: huge providers, readable plans, boring reviews, weak abstraction. Complex graphs become `for_each` / nested modules / copy-paste. Logic leaks into wrappers, Terragrunt, or generated HCL.
- Pulumi/CDK: real types, loops, shared libraries. Reviews become software reviews of a graph you cannot see until synth. Debugging is “why did this construct emit that.” Polyglot orgs split into language camps; the platform team becomes a compiler team. Provider lag and two mental models (language runtime + cloud API) show up in every outage.

Do not split the fleet. Dual IaC is the expensive outcome this memo is meant to prevent. If a team truly needs code, wrap Terraform (CDKTF, generators) behind the same state backend and policy gates — do not introduce a second source of truth.

Single biggest risk
Module explosion and unmaintainable HCL: platform abstractions that should be libraries become 15 near-copies of a 400-line module, plus `local-exec` and generated HCL. Drift between copies becomes the incident class.

Chose-wrong test (cheap)
90-day gate on the first three platform modules (network, identity, compute baseline). Fail A if either: (1) ≥3 merged changes required generating HCL, `local-exec`/`external` data, or a copy-paste of >50% of a module to vary a handful of fields; or (2) a second IaC toolchain appears in prod (Pulumi/CDK stack not going through the Terraform pipeline). Either signal means expressiveness, not process, is the bottleneck — then revisit B for that surface only, still with one state/policy plane.
