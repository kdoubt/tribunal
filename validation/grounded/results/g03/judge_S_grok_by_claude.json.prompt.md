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
I'll review the draft onboarding spec against the tree and the two prior decisions. First I'll read the spec and the referenced worker/protocol files so every claim is grounded.The spec and the worker routes, middleware, migrations, and protocol types are next so I can check fidelity, buildability, and the claim-flow attack surface.I'll verify every file, line, and mechanism the spec cites against the worker, migrations, and protocol types.Line citations in the spec look close; next I'll check MCP/capture paths, schema, and whether the pairing table can actually compose.**Framings considered:** (1) ship/no-ship readiness against the owner criteria for one unattended implementing agent; (2) claim-flow red team only; (3) citation audit of spec line pointers; (4) fidelity-only vs USER-FACT. **Picked (1)** because the brief’s criteria are fidelity, buildability on this tree, no new bearer surface, and every migration/cleanup named — with no human review between spec and build. Security and citation checks feed that gate.

Line pointers below are to `artifact/ONBOARDING_SPEC.md` (byte-identical to `docs/ONBOARDING_SPEC.md`) and the cited worker/protocol files.

---

CLAIM: The spec encodes both USER-FACT decisions (init as device-code wrapping magic-link with one Authorize click and key + `mcp_devices` enrollment; rung 0 as anonymous no-PII trial, non-bearer locator, possession-proof same-key conversion, complementary to init) but silently defaults two OWNER-CALLs (trial quota 10, free devices 3) and leaves Turnstile and raw-key vs device-token unresolved for an agent that will not get a human call.
EVIDENCE: USER-FACT owner-calls vs spec table “Items … 10 (OWNER-CALL…)” (`ONBOARDING_SPEC.md:94`), “Defaults ship as: 50/day, viewports unchanged, 3 devices” (`:240-241`), “Turnstile (OWNER-CALL: day-one vs adaptive)” (`:163`), “OWNER-CALL … raw API key vs per-device session token … UNRESOLVED” (`:202-204`); init path `:193-200`; claim possession/poll `:147-191`; “Rung 0 complements `whipnode init`” (`:22`).
CONFIDENCE: 0.86
FALSIFIER: A written owner resolution in the spec for Turnstile and key-vs-token, and the 10/3-device figures labeled as still-open rather than shipping defaults.

CLAIM: The required `accounts.email` nullability change has no complete D1 migration: email is `TEXT NOT NULL UNIQUE`, and this repo only ever `ADD COLUMN`s — SQLite/D1 cannot `ALTER` away NOT NULL without a table rebuild the spec does not give.
EVIDENCE: `apps/worker/migrations/0001_initial_schema.sql:9` `email TEXT NOT NULL UNIQUE`; spec “migration makes email nullable; SQLite UNIQUE permits multiple NULLs” (`ONBOARDING_SPEC.md:85-87`); every later migration is `ALTER TABLE … ADD COLUMN` (e.g. `0009_api_keys_override.sql:2`). EXTERNAL: SQLite `ALTER TABLE` cannot drop NOT NULL; Cloudflare D1 documents the same subset (ADD/RENAME/DROP COLUMN only).
CONFIDENCE: 0.9
FALSIFIER: A numbered migration that recreates `accounts` with nullable `email`, copies rows, and restores indexes/FKs; or a D1 statement in-tree that drops NOT NULL in place.

CLAIM: Spend-ceiling and token-purpose schema are not implementable as written: the only `trial_counters` SQL is an `UPDATE` whose `meta.changes == 0` is defined as 429, so a missing UTC-day row (every new day, and forever if no INSERT is specified) looks like cap exhaustion; no table is named for the 1/IP/24h cap; `magic_tokens` has no `purpose` column the claim/init verify split requires.
EVIDENCE: `ONBOARDING_SPEC.md:108-115` (`trial_counters` DDL + `UPDATE … WHERE day = ? AND captures < ?` / “meta.changes == 0 -> 429”); `:82` “Per-IP-hash issuance cap: 1 trial / 24h (reuse `hashIp`)” with no DDL; `:164-169` and `:247-248` “claim-specific token purpose”; `apps/worker/migrations/0002_auth_sessions.sql:5-13` (`magic_tokens` columns: id, email, token_hash, created_at, expires_at, used_at, ip_hash).
CONFIDENCE: 0.88
FALSIFIER: Spec text that names `INSERT OR IGNORE`/`UPSERT` before the conditional UPDATE, a per-IP issuance table (or equivalent), and `ALTER TABLE magic_tokens ADD COLUMN purpose …`.

CLAIM: `POST /auth/device/approve (session-authed)` cannot run for claim or init as specified: both verify paths create no account, so no `wn_session` exists, while `sessions.account_id` is `NOT NULL`.
EVIDENCE: Inventory `POST /auth/device/approve (session-authed, kind-aware)` (`ONBOARDING_SPEC.md:246-247`); claim-verify “create no account, redirect to the approve page” (`:167-169`); init “creates the account only at Authorize — not the /dashboard upsert path” (`:196-198`); `0002_auth_sessions.sql:21` `account_id TEXT NOT NULL`; session mint lives only in `auth-verify.ts:53-69` after the account upsert (`:38-51`) the spec forbids for these tokens.
CONFIDENCE: 0.87
FALSIFIER: Spec names a non-session approve authenticator (pairing id + typed code, or a purpose-built cookie) or allows a session row that does not need `accounts.id`.

CLAIM: The named 7-day unclaimed-trial cleanup will not run on any cron tick where no items are expired, because `handleScheduled` returns before later tasks when that SELECT is empty.
EVIDENCE: Spec “a NEW cleanup.ts task: revoke the key … then delete the account row” (`ONBOARDING_SPEC.md:100`); `apps/worker/src/scheduled/cleanup.ts:14-24` (`SELECT id FROM items WHERE expires_at <= ? …`; `if (!expired.results || expired.results.length === 0) { return; }`); existing token/device/upload work is already after that return (`:46-64`).
CONFIDENCE: 0.91
FALSIFIER: Spec orders the trial-expiry statements before that return, or removes the early return.

CLAIM: Wrapper-only trial issuance is not a testable worker condition: no request header/UA is named, so an implementer either mints on every unauthenticated `initialize` (raw `wn_` in an unnamed response header — a new bearer surface, plus a burned IP slot) or invents a discriminator; enrollment “via mcp.ts:333-367” also cannot run on the arriving `auth == null` request without an unstated control-flow rewrite.
EVIDENCE: `ONBOARDING_SPEC.md:62-73` (wrapper-mediated initialize; “raw `wn_` key … delivered once in a response header”; bare clients get `-32000`); `mcp.ts:291-293` (`if (!auth) { return jsonRpcError(null, -32000, …) }`) before parse; enrollment gated on `auth.type === "api_key" && isInitializeRequest(body)` (`mcp.ts:335`); header name absent from the spec; `env.ts:38-41` has no `FEATURE_ENABLE_ANON_TRIAL`.
CONFIDENCE: 0.84
FALSIFIER: Spec names the exact request signal that authorizes minting and the response header, and the mint-then-enroll sequence inside `handleMcp`.

CLAIM: Stolen 0600 file is already a complete claim credential (claim-start and converting poll both authenticate with the trial key); combining it with a leaked locator adds nothing the spec’s residual-risk paragraph does not already grant, while “a PENDING claim freezes this expiry” with no `expires_at > now` predicate lets that same key refresh freeze indefinitely (`whipnode claim` inside the 10-min TTL).
EVIDENCE: `ONBOARDING_SPEC.md:31-33` (0600 theft grants claim authority); `:153-156` claim-start “authenticates with the trial key”; `:185-191` poll “authenticated with the trial key” executes conversion; `:100` “a PENDING claim freezes this expiry”; `:162` “10-min TTL”; TTY “UX guard, not a security control” (`:151-152`).
CONFIDENCE: 0.78
FALSIFIER: Freeze SQL that requires an unexpired pending row, and a hard 7-day cap that claim-start cannot extend; or claim-start that does not treat the 0600 secret as sufficient.

CLAIM: Two state machines on one `device_codes` table are sketched, not composed: no unique partial index backs “one pending claim” / “concurrent starts can't race two emails”; init Authorize must create the account and later poll may return key material, claim Authorize must not mutate the account and must not return a key; activity snapshot “parsed” from `items.note` is lossy because a client `note` replaces the URL-bearing default and Google Doc captures store the title, not the URL (`captured_url` is response-only).
EVIDENCE: `ONBOARDING_SPEC.md:154-156, 183-191, 210-226`; `capture.ts:658-674` (`note ?? defaultNote` with `Screenshot of ${url}`); `capture.ts:494` (`note ?? \`Google Doc: ${docTitle}\``); `capture.ts:712` `captured_url: url` in JSON only; `0001_initial_schema.sql:40-58` items columns have no `captured_url`.
CONFIDENCE: 0.83
FALSIFIER: DDL with `UNIQUE(account_id) WHERE kind='claim' AND status='pending'`, kind-specific approve/poll contracts including when the init key is minted, and snapshot source that is not `note`.

---

**VERDICT INPUT**
- Q1 Fidelity: Both decisions are present; trial quota 10 and free 3-device defaults are silent owner resolutions; Turnstile and raw-key vs device-token are still open.
- Q2 Buildability: Cited line numbers for today’s bugs check out (`capture.ts:233-237`, `rate-limit.ts:20-21,70`, `mcp.ts:291-293`, `auth-verify.ts:22,38-77`, `api-keys.ts:139-142`, `types.ts:32`); schema/cleanup/approve/issuance steps are not executable as written.
- Q3 Security: Locator is non-bearer as designed; 0600 theft is the real claim credential; unnamed initialize header is a new bearer surface; pending-freeze can be gamed; TTY check is correctly non-security; poll/authorize is safe only if the atomic WHERE is real.
- Q4 Traps: Session-authed approve, missing INSERT/purpose/IP table, wrapper discriminator, kind-specific approve, note parsing, cleanup early return.

**PUBLISH VERDICT:** `blocker` — do not commit this as implementation input until email nullability has a D1 rebuild, approve auth matches “no account at verify,” and every counter/purpose/IP/cleanup path is named as SQL an agent can paste.


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}