I'll treat this as a single independent seat and check the workspace for any decision-record format I should match.This is a Tribunal seat, so I'll match the seat template and P1 brief before writing the review.I'll pull a few named sources so the evidence pointers are checkable, then write the four claims.**Framings considered:** (1) domain-native — orders are event streams, so A; (2) skill-and-requirement fit — audit + some time-travel with *this* team; (3) option value — cheaper-to-reverse door; (4) structural audit invariant — no write can skip the log. **Framing used:** (2) bounded by (3). The brief’s constraints are team skill, moderate volume, “some” temporal questions, pick now. Domain elegance is not a criterion.

---

**CLAIM:** Audit-of-change plus “order as of T” do not require the event log to be the system of record; a mutable `orders` row plus trigger-written full-row history satisfies both.

**EVIDENCE:** EXTERNAL — Fowler, *Event Sourcing* (2005): a change log is “a small gain. We could also do this by keeping a history of past ports in the ship object, or by writing to a log file whenever a ship moves. Both of these can give us an adequate history.” Distinctive ES facilities are complete rebuild and *replay of corrections* — not asked. EXTERNAL — Microsoft *Temporal table usage scenarios*: system-versioned tables are prescribed for data audit and point-in-time reconstruction. Brief: “some” temporal questions; thousands of orders/day.

**CONFIDENCE:** 0.88

**FALSIFIER:** A required first-year query that needs intent or retroactive replay of a *corrected* past event, not reconstructed column state.

---

**CLAIM:** For a six-person Postgres/REST team with no ES experience, the ES tax (projections, eventual consistency, event-schema evolution, idempotent consumers) dominates any persistence benefit at this scale.

**EVIDENCE:** EXTERNAL — Microsoft *Event Sourcing Pattern*, “might not be suitable”: systems that “naturally work well with traditional CRUD”; “consistency and real-time updates to the views of the data are required.” Same doc: materialized views are eventually consistent; changing persisted event format “can be difficult.” EXTERNAL — Fowler: the programming model is “unnatural”; “the real difficulty of Event Sourcing is hooking up to external systems that aren't centered on events” (payments, shipping, tax — an OMS’s neighbors). Brief: “no prior event-sourcing experience.”

**CONFIDENCE:** 0.85

**FALSIFIER:** This team ships a correct ES service (projections = commands, versioned events, no lost updates) on a similar calendar as CRUD+history, with no extra production incidents.

---

**CLAIM:** Throughput does not select A.

**EVIDENCE:** Brief: “thousands of orders/day, not millions.” Mechanism: even 10k orders × ~20 mutations/day ≈ 2e5 history rows/day, well inside one Postgres primary. Microsoft lists ES performance wins against CRUD *lock contention* — not this load.

**CONFIDENCE:** 0.92

**FALSIFIER:** Measured write contention or history-table cost that misses SLO under CRUD+triggers while an event store would hit it.

---

**CLAIM:** B’s load-bearing failure mode is unaudited writes if history is application-level rather than a table trigger (or equivalent in-DB enforcement).

**EVIDENCE:** Brief option B: “triggers or app-level.” EXTERNAL — Microsoft ES Context: CRUD loses history “unless there is an additional auditing mechanism.” Mechanism: an `AFTER INSERT OR UPDATE OR DELETE` row trigger on `orders` writing the old/new row to `order_history` in the same transaction cannot be skipped by ordinary DML; app-level logging can (admin SQL, missed handler, UPDATE succeeding then audit insert failing).

**CONFIDENCE:** 0.84

**FALSIFIER:** Ordinary DML that mutates `orders` without a history row while a row trigger is in place.

---

**VERDICT: B** — CRUD `orders` + in-DB append-only `order_history` (full-row snapshots; not app-level logging).

**Biggest risk:** History implemented in application code, so some write paths never record and “as of T” is silently wrong.

**Cheapest discriminating test:** Two-day spike: trigger-written full-row JSONB history; run the real support/finance “as of T” questions. Switch to A only if a required question needs *intent* or *replay-with-correction* the snapshot cannot answer. If the spike answers them, A was the overcommit.
