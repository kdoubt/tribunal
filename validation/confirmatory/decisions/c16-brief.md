# Decision C16 - shared-DB vs DB-per-tenant multitenancy

A B2B SaaS (expecting hundreds of tenants, a few very large) must pick a
multitenancy data model:

- **A) Shared database, shared schema** with a `tenant_id` column on every table
  plus row-level security.
- **B) Database-per-tenant** (an isolated database per customer).

Commit to A or B, name the single biggest risk, and give the cheapest
test/criterion that would tell them they chose wrong.
