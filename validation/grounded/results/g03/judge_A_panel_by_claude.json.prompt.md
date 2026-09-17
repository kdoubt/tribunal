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
# Verdict - G03: review a draft onboarding spec before it is committed for implementation

Built exclusively from ledger rows (ledger.md). No new arguments.

## 1. Independent agreement (agreed-r0)

Both seats, in Round 0 before any exposure, made these claims: the `accounts.email` change has no complete migration on this engine (A2, B2); the session-authed approve endpoint cannot run because neither verify path creates an account and `sessions.account_id` is NOT NULL (A4, B4); `trial_counters` is specified with an UPDATE and no row creation (A7, B3); the named 7-day cleanup sits behind `cleanup.ts`'s early return (A3, B5); the shared `device_codes` design is sketched, not composed (A5, B8); the spec preserves the ladder's core but adds product choices and leaves owner calls for an agent with no human in the loop (A1, B1). Both Round 0 publish verdicts were `blocker`. The panel concludes these.

## 2. Resolved after Round 0

**Oracle-settled (`verified`):**
- A2/B2 → 0001_initial_schema.sql:9 `email TEXT NOT NULL UNIQUE`; later migrations are ADD COLUMN only; spec :85-87 is one clause → verified.
- A4/B4 → 0002_auth_sessions.sql:21 `account_id TEXT NOT NULL`; spec :167-169, :196-198 create no account; :247 "session-authed" → verified.
- A7/B3 (counter) → spec :108-115 UPDATE only → verified. B3 (purpose) → 0002:5-13 no purpose column → verified. B3 (IP cap) → spec :82 names no table → verified.
- A3/B5 (early return) → cleanup.ts:22-24 → verified. A3 (FKs) → 0001:32,57; 0006:15-16 → FK rows verified; enforcement is EXTERNAL.
- A8 → router.ts:154-155, render.ts:33-35/56-66/118+, auth.ts:174-192, spec :245-249 → verified (route accepts any active key, outside the inventory).
- B6 (texts) → spec :66-72 header unnamed; mcp.ts:291-293; env.ts:38-41 → verified. B7 (texts) → spec :100 freeze clause has no predicate → verified. B8 (note) → capture.ts:658-674, :494, :712 → verified. A1/B1 (texts) → spec :68-72 vs :202-204; :94 label; :240-241 defaults → verified.

**Cross-examination-settled (`conceded` / `revised`):**
- A1 raw-key selected: conceded by B (R1 CONCEDE, Own-1 → 0.90). B1 "silently" for quota: conceded by A (R1 ATTACK Peer-2).
- A3 FK failure of the account delete: conceded by B (R1 CONCEDE; Own-5 → 0.93); A revised A3 → 0.95 (external premise).
- A6 post-conversion persistence: conceded by B (Own-7 → 0.86); A withdrew the re-claim walkthrough (Own-6 revised → 0.96). B7 freeze gaming: conceded by A.
- A8 render bypass: conceded by B (adds uploads as an equal unlisted path).
- B3 IP-cap semantics: conceded by A, qualified (table not necessarily new). B8 note parsing: conceded by A.

## 3. Surviving dissent

- **B6 (grok, 0.84, unrevised) vs A (R1 ATTACK Peer-5):** does the unnamed initialize response header constitute a *new bearer surface*, and does enrollment on an `auth == null` request require an unstated control-flow rewrite? A: the credential is already required and spec :66-67 names the carve-out; B: unrevised. Cheapest discriminating test: the spec author names the header and the mint-then-enroll sequence; if that text exists, B6's two clauses fall.
- **Quota 10 (minor):** A1 keeps "explicitly open" (spec :94 OWNER-CALL label); B reads it as a shipped default (:94 table, :136 wall copy). Test: whether `PLAN_LIMITS.trial` in the implemented change carries 10 without an owner sign-off.
- **Remedy for one-pending-claim (minor):** B8 requires a unique partial index; A calls it one option. Not decision-relevant to the verdict.

## Recommendation

**Mode:** `dont` (publish verdict: `blocker`). Both seats independently reached `blocker` (A1-A8, B1-B8) and Round 1 only strengthened it: every decision-relevant claim was either verified against the tree (A2, A3, A4, A7, A8, B2, B3, B4, B5) or conceded by the other seat (A1, A6, B1, B3, B7, B8). The spec should not be committed as implementation input until the migration (A2/B2), the approve authentication (A4/B4), counter initialization (A7/B3), the cleanup order and dependents (A3/B5), and pairing composition (A5/B8) are written as executable text, the raw-key delivery is reconciled with the recorded owner call (A1/B1), and the render path is brought under the trial controls (A8). The B6 dissent does not change the mode.

## Record

- Open (unexamined, not endorsed): none - every row was agreed, verified, conceded, or is in bucket 3.
- Verified vs merely agreed: verified = A2, A3 (code facts), A4, A7, A8, B2, B3, B4, B5, and the text halves of A1, B1, B6, B7, B8; agreed-only = A5/B8 composition (design judgment).
- Rounds run / stop rule hit: R0+R1; stopped: no load-bearing flip, no new claim IDs in R1.
- Transformations applied to relayed text: heading normalization of the codex memo for tooling only; ORCH-SUMMARY ledger note; neutral Peer tags, shuffled; no tallies (transformations.log).

**VERDICT INPUT**
Q1 Fidelity: the ladder's core is encoded; the spec operationally selects raw-key delivery while recording that call as unresolved, ships 3 devices (and, per one seat, quota 10) as defaults, and adds bot-score/ASN gates neither decision made.
Q2 Buildability: not buildable as written - no email-nullable migration, approve endpoint unreachable without an account, counters never initialized, cleanup behind an early return and blocked by foreign keys, no token purpose column, no per-IP cap table, pairing state machines uncomposed.
Q3 Security: the locator is non-bearer as designed; the stolen 0600 file is the real claim credential and the same key persists after conversion; the pending-claim freeze can be renewed; the render (and uploads) path sits outside the trial spend ceiling; whether the unnamed initialize header is a new bearer surface is surviving dissent.
Q4 Implementability traps: session-authed approve, missing INSERT/purpose/IP mechanism, unnamed wrapper signal and header, kind-specific approve/poll contracts, note-parsed snapshot, cleanup early return.
PUBLISH VERDICT: blocker - resolve the specification gaps above before an implementing agent builds from it.


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}