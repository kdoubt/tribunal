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
## Framing

Plausible framings: (a) fidelity audit against the two USER-FACT decisions; (b) migration/buildability audit against the tree; (c) attacker-walk of the claim protocol; (d) "is this safe to hand to one unsupervised agent." I pick **(d)**, because the stated constraint is "one implementing agent, no human review between spec and build," and because the spec already concedes its main protocol-level residual risk itself (`ONBOARDING_SPEC.md:32-34`), so unresolved forks and unnamed migrations dominate over further crypto-design debate.

---

```
CLAIM: The spec's only stated schema change for rung 0 - "migration makes email
nullable" - is not expressible as an ALTER TABLE on SQLite/D1; it needs a full
12-step table rebuild of `accounts` (5 referencing FKs, 2 indexes), and the spec
names no rebuild, no index re-creation, no FK deferral.
EVIDENCE: ONBOARDING_SPEC.md:85-87 "migration makes email nullable; SQLite UNIQUE
permits multiple NULLs" vs 0001_initial_schema.sql:9 `email TEXT NOT NULL UNIQUE`;
every migration in the tree is ADD COLUMN only (grep `ALTER TABLE` over
apps/worker/migrations returns 13 ADD COLUMN, 0 RENAME/rebuild). EXTERNAL: SQLite
ALTER TABLE supports only RENAME/ADD COLUMN/DROP COLUMN/RENAME COLUMN
(sqlite.org/lang_altertable.html); D1 rejects `PRAGMA foreign_keys=off` and offers
only `PRAGMA defer_foreign_keys` (developers.cloudflare.com/d1/sql-api/foreign-keys).
CONFIDENCE: 0.85
FALSIFIER: A D1 migration that drops NOT NULL from accounts.email without a table
rebuild, applying cleanly via `wrangler d1 migrations apply`.
```

```
CLAIM: The `npx whipnode` CLI - sole delivery channel for the rung-0 key, the 0600
file, the TTY code, and `init`/`claim`/`status` - does not exist anywhere in the
monorepo, and the spec never names creating/publishing it as work.
EVIDENCE: the only packages are apps/{web,worker} and packages/{renderers,protocol,
catalyst-client,shared}; no package.json declares a `bin`. `npx whipnode` appears
only in ONBOARDING_SPEC.md and docs/panel-review/. Spec lists it as inventory, not
work: ONBOARDING_SPEC.md:251-254.
CONFIDENCE: 0.9
FALSIFIER: A CLI package (bin entry or `apps/cli`) present in the tree.
```

```
CLAIM: Issuance is gated on `initialize` being "mediated by the npx whipnode CLI
wrapper," but the spec names no signal the worker can use to detect wrapper
mediation, so the implementer must invent one.
EVIDENCE: ONBOARDING_SPEC.md:60-61 and 71-73 ("Bare HTTP-MCP clients without the
wrapper get the existing -32000") - no header, UA, or handshake field is named;
mcp.ts:291-293 branches only on `!auth`.
CONFIDENCE: 0.85
FALSIFIER: A line in the spec naming the request field that distinguishes wrapper
from bare client.
```

```
CLAIM: The Authorize step has no defined credential: the endpoint inventory calls
approve "session-authed," but both flows explicitly create no account before
Authorize, and `sessions.account_id` is NOT NULL with an FK to accounts.
EVIDENCE: ONBOARDING_SPEC.md:247 "`POST /auth/device/approve` (session-authed,
kind-aware)" vs :169 "create no account, redirect to the approve page" and :197-199
"creates the account only at Authorize"; 0002_auth_sessions.sql:21,29
`account_id TEXT NOT NULL, FOREIGN KEY (account_id) REFERENCES accounts(id)`.
CONFIDENCE: 0.8
FALSIFIER: Spec text defining a non-session credential (e.g. a claim-scoped cookie)
for approve, or a sessions row that is legal with no account.
```

```
CLAIM: `magic_tokens` has no purpose/binding column, and the spec's migration list
never adds one - so neither the "claim-specific token purpose" nor the
token-to-claim_id binding has storage, and until it ships the existing
/auth/verify will happily consume a claim token and create the merged account the
spec forbids.
EVIDENCE: 0002_auth_sessions.sql:5-13 (columns: id, email, token_hash, created_at,
expires_at, used_at, ip_hash); ONBOARDING_SPEC.md:166-170 requires the purpose;
auth-verify.ts:39-51 upserts an account by email; the spec's named migrations are
only device_codes (:210), trial_counters (:108), email-nullable (:85).
CONFIDENCE: 0.85
FALSIFIER: A `purpose` (or device_code_id) column on magic_tokens in the tree, or
spec text naming that migration.
```

```
CLAIM: The per-IP "1 trial / 24h" cap and the claimed "pre-claim accounts store:
hashed IP" have no column or table anywhere in the spec's migration list or the
schema, leaving the primary per-identity abuse control unimplementable as written.
EVIDENCE: ONBOARDING_SPEC.md:82 "Per-IP-hash issuance cap: 1 trial / 24h (reuse
`hashIp`)" and :257 "Pre-claim accounts store: hashed IP"; accounts columns are
0001_initial_schema.sql:7-14 plus 0009_api_keys_override.sql:2 - no ip_hash;
trial_counters (:108) is day-keyed globals only.
CONFIDENCE: 0.8
FALSIFIER: An accounts.ip_hash column or a per-IP issuance table named in the spec
or present in migrations.
```

```
CLAIM: The spend-ceiling increment as written matches zero rows on the first
request of each UTC day (no row exists for the new `day` key and no INSERT is
specified), which by the spec's own rule returns 429 - and since nothing else
creates the row, trial issuance and trial capture stay 429 for the whole day.
EVIDENCE: ONBOARDING_SPEC.md:114-116 `UPDATE trial_counters SET captures =
captures + 1 WHERE day = ? AND captures < ? -- meta.changes == 0 -> 429`, with
:110-111 "append-only (never reset in place; the day key IS the reset)" and no
INSERT/upsert anywhere in the section.
CONFIDENCE: 0.75
FALSIFIER: Spec text specifying `INSERT OR IGNORE INTO trial_counters(day) VALUES(?)`
(or an upsert) before the conditional UPDATE.
```

```
CLAIM: The 7-day unclaimed-expiry cleanup task names only key revocation and
"delete the account row," with no handling of the child rows that outlive the
trial - items are tombstoned, never deleted - so the DELETE hits FK constraints
(or orphans rows) on every trial account that ever captured anything.
EVIDENCE: ONBOARDING_SPEC.md:100 "revoke the key ... then delete the account row";
cleanup.ts:189 `UPDATE items SET status = 'deleted' WHERE id = ?` (row retained as
"tombstone"); FKs to accounts at 0001_initial_schema.sql:32,57 and
0006_mcp_devices.sql:15-16. EXTERNAL: D1 enforces FK constraints by default
(developers.cloudflare.com/d1/sql-api/foreign-keys).
CONFIDENCE: 0.75
FALSIFIER: A D1 run showing `DELETE FROM accounts` succeeding with live api_keys /
mcp_devices / items rows referencing it, or spec text naming the child deletions.
```

---

**VERDICT INPUT**

- **Q1 Fidelity:** Mostly faithful, and the three USER-FACT owner-calls are all still marked OWNER-CALL (`:94`, `:170`, `:202-204`). But it adds beyond both decisions: bot-score + ASN deny-list infrastructure it admits does not exist (`:79-81`), a global spend-ceiling table, a 7-day expiry task, and a free-tier change shipped as a "default" - devices "3/account" replacing today's hardcoded 10-per-key at `mcp.ts:352`, asserting "change via constants, not code" when the quota is a SQL literal and the scope changes from per-key to per-account (`:240-241`).
- **Q2 Buildability:** Line/mechanism citations to existing code check out (capture.ts:233-237, rate-limit.ts:20-21/:70, mcp.ts:291-293/333-367/~1008, auth-verify.ts:22/38-77, api-keys.ts:139-142, types.ts:32). What is missing is everything new: the email-nullable rebuild, magic_tokens.purpose, per-IP issuance storage, child-row cleanup, the CLI package, and the approve-step credential.
- **Q3 Security:** The protocol's own reasoning is sound (conversion on the key-authed poll, typed code, never-merge) and it states the 0600-theft residual honestly at `:32-34`; the live hole is not the design but the unshipped purpose column, which leaves claim tokens redeemable at `auth-verify.ts:39-51`.
- **Q4 Traps:** Day-rollover counter, approve auth, wrapper detection, and the unresolved raw-key-vs-device-token fork (`:202-204`) that determines rung 0's delivery mechanism and init's simultaneously.

**PUBLISH VERDICT: blocker** — with no human review between spec and build, at least four required schema/cleanup changes are unnamed and the key-delivery fork is unresolved, so a single agent would have to invent the migration plan and the credential model itself.


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}