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
