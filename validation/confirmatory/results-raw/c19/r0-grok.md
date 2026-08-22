**CLAIM:** A platform standard should optimize for reviewable, constrained infra diffs, not language power. HCL plans map 1:1 to resources; Pulumi/CDK hide them in loops, classes, and generated graphs.
**EVIDENCE:** Ops-majority teams already review `plan` output and module interfaces. CDK/Pulumi PRs review code, then a second artifact (CloudFormation/TF state) that the author did not write. That split is how “clever” IaC becomes un-auditable.
**CONFIDENCE:** 0.80
**FALSIFIER:** Blind review of 20 real infra PRs: reviewers catch resource-level mistakes faster in TS/Python than in HCL.

**CLAIM:** Polyglot + general-purpose IaC fragments the standard into N languages, N libraries, N CI.
**EVIDENCE:** Pulumi/CDK sell “use the language you know.” In a polyglot org that is TypeScript *and* Python *and* Go. The platform then owns three SDKs and three review cultures. Terraform has one syntax.
**CONFIDENCE:** 0.75
**FALSIFIER:** 90-day Pulumi/CDK POC with three app teams: all converge on one language and one shared library with no forks.

**CLAIM:** Ecosystem, hiring, and vendor coverage still favor Terraform as the org-wide default.
**EVIDENCE:** Providers ship TF first; Atlantis/Spacelift/TFC, the registry, and ops hiring are TF-shaped. Pulumi often wraps those same providers; CDK is AWS-centric or emits CFN. Switching *from* TF later is easier than ripping out a language zoo.
**CONFIDENCE:** 0.85
**FALSIFIER:** Required providers (including internal ones) have equal native Pulumi coverage, and ≥50% of candidates prefer Pulumi/CDK over TF.

**CLAIM:** Growing developer involvement is served by a golden-path module, not by handing them a general-purpose language.
**EVIDENCE:** App teams typically need “instantiate service X,” not a class hierarchy. TF modules (and TF-CDK as an escape hatch) cover that without changing the control plane.
**CONFIDENCE:** 0.70
**FALSIFIER:** Timed “add a service + env-specific IAM/queue” is faster and cleaner in Pulumi/CDK for non-ops engineers than calling a published TF module.

**VERDICT: A** — Biggest risk: app developers bounce off HCL and spawn shadow Pulumi/CDK/scripts. Cheapest test: three non-ops engineers each add a realistic service using only published TF modules; if median wall-clock >1 day or 2/3 need ops to land it, A is wrong.
