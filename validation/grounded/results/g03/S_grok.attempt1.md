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
