1. **ATTACK**

- **Peer-1:** The missing per-IP issuance mechanism is substantiated, but a new table is not necessarily required: `mcp_devices` already stores `ip_hash` and `created_at` (`0006_mcp_devices.sql:7–8`). The defect is missing durable counting semantics, atomic enforcement, and retention rules. The counter-initialization and token-purpose findings stand as settled.
- **Peer-2:** “Silently defaults” overstates the quota finding: `ONBOARDING_SPEC.md:94` explicitly labels 10 an OWNER-CALL. Conversely, “leaves raw-key vs device-token unresolved” understates the contradiction: :68–72 operationally mandates raw-key delivery and `X-API-Key`, while :202–204 requires both paths to follow the unresolved choice. Three devices is explicitly a shipping default (:240–241), but USER-FACT does not authorize that additional decision.
- **Peer-3:** The stolen file supplies the trial-side credential; the attacker still needs mailbox verification and authorization, which they can satisfy using their own unused mailbox. The acknowledged theft risk does **not** establish a 24-hour bound: same-key conversion (:189) preserves the thief’s access afterward (`auth.ts:174–192`). Indefinite freeze is a credible missing-policy exploit, not demonstrated executable behavior. Even an expiry predicate would not prevent repeated timely renewal without an absolute deadline.
- **Peer-4:** A unique partial index is one solution, not a necessary condition for serialized starts. An index covering only `pending` also leaves `verified` and `approved` replacement semantics unresolved. Kind-specific approve/poll behavior is partially specified (:183–199, :222–226); atomic composition is what remains missing. The note-parsing objection is established: client notes replace defaults, and Google Docs default to titles (`capture.ts:494,674`).
- **Peer-5:** The wrapper signal and response-header name are genuinely unspecified. But “unstated control-flow rewrite” is too strong: :66–67 explicitly identifies the early rejection and requires an initialize carve-out. The missing detail is how issuance constructs authentication context and reaches enrollment. Delivering an already-required credential in a header does not, by itself, prove an additional bearer capability or leak.

2. **CONCEDE**

The final answer must retain missing per-IP issuance semantics, unreliable activity snapshots, and unspecified wrapper/header contracts alongside the settled blockers. TTY is explicitly bypassable; leaked locators add no authority to an attacker already holding the trial key.

The rendering bypass remains independently decisive: `render.ts:33–66,118–141` admits authenticated rendering and allocates Browser resources outside the specified trial controls. Account deletion also needs explicit dependent-row handling: revocation changes statuses, while foreign-key references remain (`api-keys.ts:136–142`; `0001_initial_schema.sql:32,57`).

3. **REVISE**

- **Own-3 — new text:** “Under enforced foreign keys, revoke-then-delete fails while key/device/item references remain; the spec also lacks a complete independent cleanup lifecycle.” **Confidence: 0.98 → 0.95.** Enforcement is an external premise, not established by the repository alone.
- **Own-6 — new text:** “File theft supplies trial-side claim authority, and unchanged credentials retain account access after legitimate or attacker-driven conversion; item TTL does not bound that access.” **Confidence: 0.94 → 0.96.**
- **Own-1/2/4/5/7/8:** No revision; their artifact evidence survives. Midnight adjacency consumes two calendar-day budgets rather than defeating either cap.

4. **VERDICT INPUT**

Q1 Fidelity: Preserve the ladder; reconcile credential delivery and explicitly resolve owner calls and added product choices.  
Q2 Buildability: Block on migrations, browser authentication, atomic pairing completion, counters, and cleanup.  
Q3 Security: Close rendering bypass, continuing stolen-key access, and renewable expiry freezes.  
Q4 Implementability: Specify wrapper contracts, snapshot provenance, transition predicates, and retry behavior.  
PUBLISH VERDICT: **blocker** — the implementing agent would otherwise invent consequential authentication, database, and enforcement behavior.
