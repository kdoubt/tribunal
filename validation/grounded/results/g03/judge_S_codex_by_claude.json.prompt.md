You are scoring one review memo against a sealed rubric. You are not a participant; do not add your own review.
Score ONLY from the memo text. Quote the memo span that earns each point; if no span supports a point, the score is 0.

=== SEALED RUBRIC ===
# G03 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (one must_catch was MISSED by the original three-seat panel)
- **correct_call:** **ship-after-fixes.** The staged spec is v2, i.e. it
  already incorporates the blockers the original review found (conversion on
  the key-authenticated poll, claim-specific magic-token purpose, `kind`
  discriminator on `device_codes`, rung-0 key via the CLI wrapper only,
  UTC-day atomic counters, PLAN_LIMITS fallback closed). What it still lacks
  is a complete migration plan: making `accounts.email` nullable on SQLite/D1
  is a full table rebuild, and the spec's new 7-day cleanup query needs an
  index that the rebuild does not carry.
- **oracle:** the spec was implemented from v3 (PR #3, merged 2026-08-20,
  5ab30b0) and launched the same day; four post-build review passes found
  implementation defects and none contradicted the spec; the one thing the
  spec review "could not see" was the migration splice: commit cf8c602
  (migration 0013, "full FK-closure rebuild for nullable accounts.email")
  dropped `idx_accounts_trial_unclaimed`, restored by 0470f82 (migration
  0014) the same day.
- **correct fix:** the spec's "Prerequisite bugfixes" and "Shared pairing
  table" sections must be joined by a migration section stating: (a)
  `accounts.email` cannot be altered in place (`0001_initial_schema.sql:9`
  `email TEXT NOT NULL UNIQUE`; SQLite has no ALTER COLUMN), so the change is
  a create-new/copy/drop/rename rebuild that must recreate **every** existing
  `accounts` index (`0001_initial_schema.sql:16-18`: `idx_accounts_email`,
  partial `idx_accounts_stripe`) and every FK that references `accounts`;
  (b) the new cleanup sweep (`ONBOARDING_SPEC.md:100`: revoke trial keys
  where `plan='trial' AND email IS NULL AND created_at < now-7d`) needs a
  partial index on `accounts(created_at) WHERE plan='trial' AND email IS NULL`
  created in the same migration.
- **must_catch:**
  1. **[MISSED by the original panel; scored on the omission alone, not on
     what a later migration did]** The email-nullable change is a
     SQLite table rebuild, and the spec names no migration plan for it, no
     index recreation, and no index for its own 7-day cleanup query
     (`ONBOARDING_SPEC.md:85-87` "migration makes email nullable";
     `:100` the cleanup task; `0001_initial_schema.sql:7-18`). A reviewer who
     asks "what does the migration look like on D1" finds it.
  2. Fidelity: the spec records the three owner calls as open OWNER-CALLs
     (`ONBOARDING_SPEC.md:263-285` Decision log), which is correct; a claim
     that it "silently resolved" the raw-key or quota dissent is wrong for v2.
  3. Buildability of the claim flow rests on a claim-specific magic-token
     purpose because the existing verify handler upserts a `plan='free'`
     account by email and 302s to `/dashboard`
     (`apps/worker/src/routes/auth-verify.ts:38-50,71-77`); the spec says so
     (`ONBOARDING_SPEC.md:161-168`) and a reviewer must confirm the code
     agrees.
  4. `device_codes` needs a `kind` discriminator because init poll returns key
     material and claim poll executes conversion (`ONBOARDING_SPEC.md:206-226`);
     any spec without it does not compose.
  5. Prerequisite bugfixes are real: capture path skips the per-IP limiter
     (`capture.ts:233-236`), MCP internal Request drops `CF-Connecting-IP`
     (`mcp.ts:1008-1013`), limiter fails open (`rate-limit.ts:20-21`),
     `PLAN_LIMITS` fallback (`rate-limit.ts:70`); the spec lists all four
     (`ONBOARDING_SPEC.md:38-57`).
  6. The rung-0 raw key must never appear in RPC/tool text (transcript bearer);
     the spec routes it via the CLI wrapper only (`ONBOARDING_SPEC.md:243-253`).
  7. Revoked-key-mid-flow: approve/poll must require the trial key to still be
     `active` (`api-keys.ts:134-147` cascade revokes devices); the spec's
     claim protocol step 1 says so (`ONBOARDING_SPEC.md:149-155`).
- **landmine (confident wrong answers):** "blocker: conversion on browser
  Authorize is a bearer regression" (that was v1; v2 converts on the poll -
  a reviewer citing this without reading the staged text is reviewing from
  memory); "the spec resolves the quota dissent" (it lists it as OWNER-CALL);
  "no migration needed, just relax the constraint" (SQLite cannot).
- **unverifiable:** the four post-build implementation defects (ae6adba,
  7177649, 6f9dda5, 965699c) are not in the staged tree; do not score them.
- **outcome_source:** `2026-08-19-onboarding-spec-review/retro.md`
  (`missed_entirely: migration splice: 0013 rebuild dropped
  idx_accounts_trial_unclaimed, restored by 0014 the same day`; T1 outcome;
  verdict_held yes); `codex-rev.md` findings 1-8; `grok-rev.md` findings 1-8;
  whipnode repo commits cf8c602 and 0470f82 (`0014_trial_unclaimed_index.sql`
  comment: "lost when that section was replaced by the FK-closure rebuild").


=== MEMO (author unknown) ===
Plausible framings are decision fidelity, implementation readiness, and protocol security. I choose **implementation readiness**, using fidelity and attacker walkthroughs as acceptance checks, because the brief sends this document directly to one implementing agent.

Pointers below are relative to `artifact/`; `S` means `ONBOARDING_SPEC.md`, `W` means `apps/worker/src`, and `M` means `apps/worker/migrations`.

**CLAIM 1:** The draft preserves the central ladder but adds product decisions and operationally selects raw trial keys despite calling that choice unresolved.  
**EVIDENCE:** USER-FACT; S:22 preserves complementary init, S:185–189 preserves same-key conversion; S:68–72 mandates raw-key delivery and `X-API-Key`, contradicting S:202–204; S:79–82 adds mandatory bot-score/ASN gates; S:236–241 introduces additional free-tier owner calls and ships three devices/account. Trial quota and Turnstile remain explicitly open at S:94 and S:163.  
**CONFIDENCE:** 0.96  
**FALSIFIER:** A supplied decision authorizes those additions and reconciles mandatory raw-key delivery with the unresolved credential choice.

**CLAIM 2:** The required schema changes lack a complete migration plan for the existing database.  
**EVIDENCE:** S:85–87 says only “migration makes email nullable”; M/0001_initial_schema.sql:9 declares `email TEXT NOT NULL UNIQUE`, :16–18 adds indexes, and M/0009_api_keys_override.sql:2 adds a column that must survive any rebuild. S:164–169 requires token purposes and pairing association, but M/0002_auth_sessions.sql:5–13 contains neither. S:108–115 and :210–220 provide table sketches without an ordered migration plan. EXTERNAL: [D1 foreign-key documentation](https://developers.cloudflare.com/d1/sql-api/foreign-keys/) specifies enforced foreign keys and transaction-scoped deferral, rather than disabling enforcement.  
**CONFIDENCE:** 0.97  
**FALSIFIER:** An artifact section supplies executable migrations preserving existing data, columns, indexes, and references while adding token-purpose/pairing bindings and both new tables.

**CLAIM 3:** The specified seven-day account deletion fails against existing foreign keys, and the cleanup lifecycle is incomplete.  
**EVIDENCE:** S:100 prescribes revoking the key and then deleting the account; W/routes/api-keys.ts:136–143 only updates key/device statuses. M/0001_initial_schema.sql:32 and :57 retain account references, and M/0006_mcp_devices.sql:15–16 retains account/key references. W/scheduled/cleanup.ts:188–191 retains item tombstones; :22–24 returns before subsequent maintenance when no items expire. S:100 freezes expiry for pending claims, while S:210–220 supplies expiry fields without specifying a pairing expiry/purge task. EXTERNAL: [D1 foreign-key enforcement](https://developers.cloudflare.com/d1/sql-api/foreign-keys/).  
**CONFIDENCE:** 0.98  
**FALSIFIER:** A specified cleanup order removes or detaches dependents, expires/purges pairings, and executes independently of the expired-item batch.

**CLAIM 4:** First-time init cannot reach the specified session-authenticated Authorize endpoint using the browser authentication mechanism described.  
**EVIDENCE:** S:195–198 postpones account creation until Authorize; S:247 requires Authorize to be “session-authed.” M/0002_auth_sessions.sql:21,29 requires every existing session to reference an account; W/routes/auth-verify.ts:64–74 creates that account-bound session cookie. S:164–169 describes mailbox verification and redirect but no temporary browser credential or its binding to the pairing.  
**CONFIDENCE:** 0.96  
**FALSIFIER:** A described pre-account browser authentication mechanism binds the verified mailbox, pairing, purpose, and Authorize request without making the locator a bearer credential.

**CLAIM 5:** The shared pairing design does not fully specify atomic completion, identity binding, or expiry enforcement, despite naming states and revoked-key checks.  
**EVIDENCE:** S:185–191 specifies one account UPDATE checking approved/unconsumed pairing and active key/device, but does not specify how pairing consumption commits with conversion or include `expires_at` in its listed assertions. S:210–220 has no key/device identifiers identifying the device whose activity must be checked. S:154–156 promises one pending claim; S:218 also introduces `verified` and `approved` without defining whether replacement invalidates those states.  
**CONFIDENCE:** 0.91  
**FALSIFIER:** Concrete transition predicates and transactional operations identify the bound account/key/device, enforce TTL, define replacement for every live state, and handle concurrent polls and retries consistently.

**CLAIM 6:** Credential-file theft can retain authority after conversion, so the stated 24-hour blast-radius rationale does not bound continuing account access.  
**EVIDENCE:** S:31–33 acknowledges file theft; S:151–161 permits key-authenticated claim-start and explicitly treats TTY as bypassable; S:189 preserves the same key. W/middleware/auth.ts:174–192 continues accepting an active key after the account changes plan, and W/routes/account-items.ts:19–20 scopes history to that account. A thief can start a replacement claim, obtain its code, verify their own unused mailbox, authorize, and poll; the original CLI continues using that same account.  
**CONFIDENCE:** 0.94  
**FALSIFIER:** Conversion requires an independent possession factor unavailable from the stolen file, or invalidates the stolen credential’s continuing authority.

**CLAIM 7:** The UTC counter design prevents an in-place reset race but omits creation of each day’s counter row.  
**EVIDENCE:** S:108–115 defines a day-keyed table and only `UPDATE ... WHERE day = ? AND captures < ?`, with zero changes mapped to 429. There is no daily insert/upsert in that section. Its day boundary also permits one budget immediately before midnight and another afterward; that follows the advertised calendar-day policy at S:110–111 and :136, rather than exceeding either day’s cap.  
**CONFIDENCE:** 0.97  
**FALSIFIER:** A specified atomic initialization/upsert admits the first allocation of a new UTC day while concurrent allocations remain capped.

**CLAIM 8:** The enforcement inventory leaves an authenticated Browser-rendering path outside the trial spend and viewport controls.  
**EVIDENCE:** S:95 and :245–249 name capture validation, while W/routes/router.ts:154–155 exposes `/api/v1/render`. W/routes/render.ts:33–35 accepts authenticated callers, :56–66 permits six viewports and full-page rendering, and :118–141 allocates storage and launches Browser without a quota check. W/middleware/auth.ts:174–192 accepts active API keys without excluding trial plans.  
**CONFIDENCE:** 0.96  
**FALSIFIER:** The spec explicitly rejects trial rendering or requires the shared trial budget, concurrency reservation, and viewport restrictions before this path allocates resources.

**VERDICT INPUT**

Q1: Core decisions survive, but added defaults and the unresolved/raw-key contradiction require correction (claim 1).  
Q2: Migration, cleanup, browser approval authentication, and pairing completion are insufficiently specified (claims 2–5).  
Q3: Locator-only conversion is barred by S:147–149; stolen-key persistence and uncovered rendering remain material risks, while UTC boundary bursts follow the stated policy (claims 6–8).  
Q4: The implementing agent must currently invent consequential authentication, transaction, expiry, and counter-initialization behavior (claims 2–5, 7).

**PUBLISH VERDICT: blocker** — Resolve these specification gaps before implementation proceeds without human review.


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}