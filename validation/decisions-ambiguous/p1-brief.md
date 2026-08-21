# Decision P1 - event sourcing vs CRUD-with-audit-log

A team of ~6 engineers is building a new **order-management service** for a
mid-market e-commerce company. Requirements: a reliable **audit trail** of every
change to an order, the ability to answer some **"what did this order look like
at time T"** temporal questions, moderate throughput (thousands of orders/day,
not millions), and a team that is strong on Postgres/REST but has **no prior
event-sourcing experience**. They must pick a persistence approach now:

- **A) Event sourcing** - orders are a log of immutable events; current state is a
  projection. Native audit + time-travel; new paradigm for the team.
- **B) CRUD + explicit audit log** - a normal mutable `orders` table plus an
  append-only `order_history`/audit table (triggers or app-level), snapshots for
  temporal queries as needed.

There is no industry consensus; both ship successful products. **Commit to A or
B**, name the single biggest risk of your choice, and give the cheapest test or
criterion that would tell this team they picked wrong.
