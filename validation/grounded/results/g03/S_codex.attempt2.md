Plausible framings: fidelity to the adopted decisions; completeness as an implementation contract; or protocol security. I choose implementation-contract completeness, because the brief permits no human review between this spec and the build.

Pointers below are relative to `artifact/`; `S` = `ONBOARDING_SPEC.md`, `W` = `apps/worker/src`, and `M` = `apps/worker/migrations`.

1. **CLAIM:** The core ladder is preserved, but mandatory raw-key handling effectively resolves an expressly unresolved owner call, while the three-device free-tier default adds a decision absent from USER-FACT.  
   **EVIDENCE:** USER-FACT; S:22,147–149,189–190,195–204; S:70–72 mandates storing the raw key and supplying `X-API-Key`; S:240–241 says “Defaults ship as: 50/day, viewports unchanged, 3 devices”; S:94,163 preserves the quota and Turnstile owner calls.  
   **CONFIDENCE:** 0.96  
   **FALSIFIER:** An explicit owner adoption of raw-key delivery and three free devices, or revised requirements that consistently support either credential choice.

2. **CLAIM:** The required schema changes lack a complete migration plan, particularly for preserving the populated accounts table while removing email’s NOT NULL constraint and adding purpose-bound pairing tokens.  
   **EVIDENCE:** S:85–87 only says “migration makes email nullable”; S:164,196 requires token purposes; S:210–220 sketches `device_codes`. M/0001_initial_schema.sql:9,16–18 defines the constraint and indexes; M/0009_api_keys_override.sql:2 adds a column that must survive; M/0002_auth_sessions.sql:5–13 has neither purpose nor pairing binding. **EXTERNAL:** [D1 foreign-key documentation](https://developers.cloudflare.com/d1/sql-api/foreign-keys/) specifies enforced foreign keys and transaction-scoped deferral, rather than disabling enforcement.  
   **CONFIDENCE:** 0.98  
   **FALSIFIER:** A migration sequence specifying data preservation, indexes, dependent foreign keys, token bindings, and validation against an already-populated D1 database.

3. **CLAIM:** The prescribed “revoke key, then delete account” cleanup fails against existing foreign keys, and pairing expiration needs explicit scheduling before it can safely govern the cleanup freeze.  
   **EVIDENCE:** S:100 prescribes that sequence and freezes expiry for “PENDING” claims; W/routes/api-keys.ts:136–143 only updates statuses; M/0001_initial_schema.sql:32,57 and M/0006_mcp_devices.sql:15–16 retain references; W/scheduled/cleanup.ts:188–191 retains item tombstones. S:218–219 lists pairing states and expiry without a cleanup transition. W/scheduled/cleanup.ts:22–24 returns before later maintenance when no items expire. **EXTERNAL:** [D1 foreign-key enforcement](https://developers.cloudflare.com/d1/sql-api/foreign-keys/).  
   **CONFIDENCE:** 0.98  
   **FALSIFIER:** An explicit child-row deletion/detachment policy plus independently scheduled pairing expiry and account cleanup, including verified/approved claims and the no-expired-items case.

4. **CLAIM:** Browser approval has an authentication bootstrap gap: the spec requires session authentication but supplies no pairing-scoped browser session before account creation.  
   **EVIDENCE:** S:247 specifies “session-authed”; S:168–169 says verification writes `verified_email`, creates no account, and redirects; S:196–198 postpones init account creation until Authorize. Existing sessions require `account_id` in M/0002_auth_sessions.sql:21,29; W/middleware/auth.ts:204–209 resolves sessions by joining accounts.  
   **CONFIDENCE:** 0.96  
   **FALSIFIER:** A specified, expiring browser authorization context bound to the verified mailbox, pairing, and purpose, including its issuance and validation without prematurely creating an account.

5. **CLAIM:** Although revoked-key rechecks are explicitly required, the shared pairing protocol leaves conversion/consumption atomicity, original-device binding, and stale-flow exclusion insufficiently specified.  
   **EVIDENCE:** S:185–191 updates account email/plan and checks pairing, key, and device; S:210–220 provides no key/device identifier; S:225–226 requires atomic transitions without specifying their coupling. S:154–156 invalidates prior “pending” claims although S:218 also permits `verified` and `approved`; the conversion predicate at S:187–188 omits `expires_at`. W/middleware/auth.ts:186–192 returns key identity without device identity.  
   **CONFIDENCE:** 0.91  
   **FALSIFIER:** Concrete conditional statements and transaction boundaries that bind the original credentials, consume the pairing with conversion, enforce expiry, and reject old verified/approved flows under concurrent start, authorize, poll, and revoke requests.

6. **CLAIM:** Stolen credential-file access is not bounded to anonymous trial data: an attacker can start a replacement claim, and the stolen key remains valid after a legitimate same-key conversion.  
   **EVIDENCE:** S:31–33 describes the claimed blast-radius bounds; S:151–156 explicitly treats TTY as bypassable and allows key-authenticated claim-start; S:189 preserves the same key; S:125–127 exempts converted accounts from rung-0 controls. W/middleware/auth.ts:174–175 accepts active keys on active accounts.  
   **CONFIDENCE:** 0.95  
   **FALSIFIER:** A specified independent possession factor or conversion-time credential invalidation that makes the stolen file unusable afterward. Locator secrecy and the existing TTY requirement would not falsify this claim.

7. **CLAIM:** The enforcement inventory leaves a concrete trial spend-ceiling bypass through the existing authenticated render endpoint.  
   **EVIDENCE:** S:95,102–104,249 names capture validation; W/routes/router.ts:154–155 exposes `/api/v1/render`; W/routes/render.ts:33–35 accepts authenticated callers, :56–66 permits multiple viewports/full-page, and :111–141 reaches browser allocation from supplied HTML without a daily quota, trial counter, or trial-plan rejection.  
   **CONFIDENCE:** 0.95  
   **FALSIFIER:** An explicit requirement to deny trial rendering or apply the same atomic budget, concurrency, viewport, and kill-switch checks before every render allocation.

8. **CLAIM:** UTC-day counters avoid resetting a live counter, but the specified UPDATE has no first-use row initialization and permits adjacent days’ budgets to be spent around midnight.  
   **EVIDENCE:** S:108–115 defines day-keyed rows and only `UPDATE ... WHERE day = ? AND captures < ?`, mapping zero changes to 429; S:136 advertises midnight reset.  
   **CONFIDENCE:** 0.97  
   **FALSIFIER:** A specified atomic day-row creation/increment sequence resolves initialization; an additional rolling-window limit would falsify the midnight-burst observation.

**VERDICT INPUT**

Q1 — Core fidelity holds, but credential-choice contradictions and added free-device policy require reconciliation (claim 1).  
Q2 — Migration, deletion, browser-session bootstrap, and pairing transitions are incomplete (claims 2–5).  
Q3 — Locator-only conversion is rejected, but stolen-key persistence and render-budget bypass remain; midnight allows bounded adjacent-day bursts (claims 6–8; S:147–149).  
Q4 — One implementing agent must currently invent consequential authentication, transaction, cleanup, and counter behavior (claims 2–5,8).

**PUBLISH VERDICT: blocker** — Resolve these contract gaps before implementation, especially migration/deletion semantics, approval authentication, pairing atomicity, and coverage of browser-spending endpoints.
