Plausible framings are decision fidelity, implementation readiness, and protocol security. I choose implementation readiness: the USER-FACT constraint—one implementing agent, no intervening human review—makes unresolved mechanics consequential.

All paths below are relative to `artifact/`; `S` means `ONBOARDING_SPEC.md`, `W` means `apps/worker/src`, and `M` means `apps/worker/migrations`.

1. **CLAIM:** The draft preserves the ladder but adds unratified requirements and contradicts the unresolved credential choice.
   
   **EVIDENCE:** USER-FACT; `S:22` preserves complementary init; `S:185-190` preserves same-key conversion; `S:62-81` adds wrapper-only issuance and mandatory bot-score/ASN gates; `S:240-241` selects three devices; `S:68-72` requires raw-key delivery despite `S:202-204` leaving that choice unresolved; `S:195-200` does not explicitly carry forward init’s one-key-plus-device-enrollment requirement. Trial quota and Turnstile remain labeled OWNER-CALL (`S:94,163`).
   
   **CONFIDENCE:** 0.94
   
   **FALSIFIER:** An owner-approved specification reconciles the credential alternatives, explicitly restores init enrollment, and ratifies the added restrictions.

2. **CLAIM:** The schema requirements lack a complete migration plan for the existing D1 database.
   
   **EVIDENCE:** `S:85-88` says “migration makes email nullable”; `M/0001_initial_schema.sql:9` declares `email TEXT NOT NULL UNIQUE`; `M/0009_api_keys_override.sql:2` adds an account column that migration must preserve. `S:164-169` requires token-purpose separation and pairing association, absent from `M/0002_auth_sessions.sql:5-13`; `S:108-115,210-220` supplies sketches, not ordered migrations. EXTERNAL: [D1 foreign-key documentation](https://developers.cloudflare.com/d1/sql-api/foreign-keys/) states enforcement cannot simply be disabled during migrations.
   
   **CONFIDENCE:** 0.97
   
   **FALSIFIER:** A specified migration sequence preserves populated accounts, indexes, override values, and references; permits multiple NULL emails; and creates pairing, token-binding, and counter structures with their constraints.

3. **CLAIM:** The prescribed trial cleanup cannot delete accounts as written and leaves pairing expiration insufficiently specified.
   
   **EVIDENCE:** `S:100`: “revoke the key … then delete the account row; a PENDING claim freezes this expiry.” Revocation only updates statuses (`W/routes/api-keys.ts:134-143`); account references remain (`M/0001_initial_schema.sql:32,57`; `M/0006_mcp_devices.sql:15`). Item cleanup retains tombstones (`W/scheduled/cleanup.ts:188-191`), and the scheduler returns before subsequent maintenance when no items expire (`:22-24`). `S:219` gives pairing expiry timestamps without naming an expiration/purge task. EXTERNAL: [D1 enforces these references](https://developers.cloudflare.com/d1/sql-api/foreign-keys/).
   
   **CONFIDENCE:** 0.98
   
   **FALSIFIER:** An explicit dependency cleanup policy deletes an expired trial successfully, runs without expired items, and prevents expired pending claims from indefinitely freezing account expiry.

4. **CLAIM:** Fresh-browser approval has an authentication bootstrap gap.
   
   **EVIDENCE:** `S:247` requires session-authenticated approval; `S:168-169` makes claim verification write only `verified_email`; `S:195-198` defers init account creation until Authorize. Existing session creation requires an account (`W/routes/auth-verify.ts:53-68`; `M/0002_auth_sessions.sql:21,29`), and `resolveSession` joins that account (`W/middleware/auth.ts:204-210`). No pairing-scoped browser authentication is specified.
   
   **CONFIDENCE:** 0.97
   
   **FALSIFIER:** A documented authentication mechanism lets a browser with no existing cookie authorize either kind while binding mailbox proof to that exact pairing and keeping the locator non-authorizing.

5. **CLAIM:** Although revoked-key checks are specified, the shared pairing design does not fully specify atomic conversion, consumption, or credential binding.
   
   **EVIDENCE:** `S:185-191` requires one account UPDATE that checks approval, active key/device, and email uniqueness; it never states how that UPDATE also consumes the pairing. `S:210-220` contains no key/device identifiers, and `S:154-156` invalidates “pending” claims despite separately enumerated `verified` and `approved` states (`:218`). `S:225-226` promises atomic transitions without their predicates.
   
   **CONFIDENCE:** 0.93
   
   **FALSIFIER:** Explicit transitions and transaction statements bind the initiating credentials, enforce expiry, freeze the approved email, handle supersession across live states, and demonstrate correct outcomes under concurrent starts, authorization, revocation, and polls.

6. **CLAIM:** The draft understates stolen-file exposure because same-key conversion preserves attacker access beyond the trial’s 24-hour item lifetime.
   
   **EVIDENCE:** `S:31-33` acknowledges file theft but describes a blast radius capped partly by 24-hour TTL; `S:151-156` permits key holders to start replacement claims and explicitly treats TTY checks as bypassable; `S:189` retains the same key after conversion. API-key authentication checks active status without a key expiration (`W/middleware/auth.ts:143-175`). A leaked locator alone remains insufficient under `S:147-149`.
   
   **CONFIDENCE:** 0.95
   
   **FALSIFIER:** A specified independent possession factor or post-claim credential restriction prevents the copied key from authenticating after legitimate conversion; otherwise the residual-risk statement must acknowledge continuing account access.

7. **CLAIM:** UTC-day counters avoid an in-place reset race but leave initialization and the separate rolling issuance cap unspecified.
   
   **EVIDENCE:** `S:108-115` defines a day-keyed table and only an UPDATE, with zero changed rows producing 429; no daily-row insertion is specified. `S:82` requires one issuance per IP per 24 hours, while the counter schema has no IP dimension. Separate day keys also permit spending each adjacent day’s allowance across midnight (`S:109-111`), consistent with calendar-day—not rolling-day—budgeting.
   
   **CONFIDENCE:** 0.96
   
   **FALSIFIER:** Specified atomic row initialization and a durable IP issuance ledger pass concurrent first-request and midnight tests, with the calendar-boundary burst explicitly accepted or constrained.

8. **CLAIM:** The enforcement inventory leaves an existing browser-render route available to trial credentials without the proposed capture restrictions.
   
   **EVIDENCE:** `S:95,102-104,245-249` names capture validation but omits render enforcement; `W/routes/router.ts:154-155` exposes `/api/v1/render`; `W/routes/render.ts:33-35` accepts any authenticated identity, `:56-66` permits up to six viewports and full-page rendering, and `:141` launches the browser without a preceding quota check.
   
   **CONFIDENCE:** 0.97
   
   **FALSIFIER:** The spec explicitly denies trial rendering or applies the same atomic spend reservation, concurrency, viewport, and kill-switch controls before browser allocation.

**VERDICT INPUT**

Q1: Partial fidelity; reconcile credential choice and ratify additions (claim 1).

Q2: Not build-ready; migrations, cleanup, and approval authentication are incomplete (claims 2–4).

Q3: Locator-only protection is stated, but persistent stolen-key authority and an uncovered spending path remain (claims 5–8).

Q4: An implementer must invent consequential transition, binding, and counter behavior (claims 4–7).

**PUBLISH VERDICT: blocker** — Resolve the executable-contract gaps before handing this spec to an implementing agent without human review.
