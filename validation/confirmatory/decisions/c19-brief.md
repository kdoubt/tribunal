# Decision C19 - Terraform vs a general-purpose IaC (Pulumi/CDK)

A platform team standardizing infrastructure-as-code for a polyglot org (mostly ops-leaning engineers, growing app-developer involvement) must pick a tool:

- **A) Terraform** (HCL, declarative, huge provider ecosystem).
- **B) A general-purpose-language IaC** (Pulumi / CDK - real code, loops/abstractions).

Commit to A or B, name the single biggest risk, and give the cheapest test/criterion that would tell them they chose wrong.
