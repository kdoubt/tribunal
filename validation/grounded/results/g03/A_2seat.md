# A_2seat = union of two isolated Round 0 memos, no ledger, no Round 1

## Memo 1 (verbatim)

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

-----

## Memo 2 (verbatim)

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
