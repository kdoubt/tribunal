# g03 oracles (orchestrator-run, non-LLM, 2026-09-16)

Method: every `file:line` in both memos opened with tools/grounding.py (S_codex.grounding.json, S_grok.grounding.json)
and then by hand. "verified" = the cited text settles the stated fact by inspection; nothing else is rated.

Resolver note: codex's `W/middleware/auth.ts:174-192` was reported MISSING by the resolver because the basename search
picked a different `auth.ts`; opened by hand: apps/worker/src/middleware/auth.ts has 254 lines; :174-192 is the api-key
acceptance path (`if (!row || row.status !== "active" || row.account_status !== "active") return null;` then returns
`{type:"api_key", ... plan: row.plan}` with no plan-based exclusion). Pointer resolves.

| pointer | opens | what the text says |
|---|---|---|
| ONBOARDING_SPEC.md:22 | yes | "Rung 0 complements `whipnode init`; it never replaces it." |
| ONBOARDING_SPEC.md:66-72 | yes | key "delivered once in a response header consumed by the npx whipnode stdio wrapper, which writes the 0600 credential file and thereafter supplies `X-API-Key`"; header not named |
| ONBOARDING_SPEC.md:79-82 | yes | bot-score >= BOT_SCORE_MIN (OWNER-CALL), ASN deny-list "neither exists in-repo today"; "Per-IP-hash issuance cap: 1 trial / 24h (reuse `hashIp`)" - no table/DDL named |
| ONBOARDING_SPEC.md:85-87 | yes | "create accounts row (plan='trial', email=NULL - migration makes email nullable; ...)" - one clause, no migration text |
| ONBOARDING_SPEC.md:94 | yes | "Items ... 10 (OWNER-CALL; codex-seat dissent: 3 until billing telemetry)" - labelled OWNER-CALL, value 10 in the shipping table |
| ONBOARDING_SPEC.md:100 | yes | "7 days - a NEW cleanup.ts task: revoke the key (existing cascade revokes devices, api-keys.ts:139-142), then delete the account row; a PENDING claim freezes this expiry" |
| ONBOARDING_SPEC.md:108-115 | yes | trial_counters DDL + one `UPDATE ... WHERE day = ? AND captures < ?  -- meta.changes == 0 -> 429`; no INSERT/UPSERT in the section |
| ONBOARDING_SPEC.md:147-149 | yes | "A leaked locator or a phished click can therefore never convert anything by itself." |
| ONBOARDING_SPEC.md:151-156 | yes | TTY "a UX guard, not a security control"; "Claim-start authenticates with the trial key (key must be status='active'); it invalidates any prior pending claim" |
| ONBOARDING_SPEC.md:157-159 | yes | snapshot "parsed at claim time - there is no `captured_url` column; items carry the URL only in `note`" |
| ONBOARDING_SPEC.md:163-169 | yes | Turnstile OWNER-CALL; claim-verify "create no account, redirect to the approve page" |
| ONBOARDING_SPEC.md:185-191 | yes | poll "authenticated with the trial key" executes conversion as one atomic UPDATE re-asserting approved+unconsumed, plan='trial', key+device active, email unclaimed; "Same key" |
| ONBOARDING_SPEC.md:195-198 | yes | init "creates the account only at Authorize - not the /dashboard upsert path" |
| ONBOARDING_SPEC.md:202-204 | yes | "OWNER-CALL (recorded as UNRESOLVED init-panel dissent, not adopted): raw API key vs per-device session token" |
| ONBOARDING_SPEC.md:210-224 | yes | device_codes sketch: kind, hashes, account_id, status enum, expires_at "10 min both kinds"; no UNIQUE partial index; "Only the init poll may ever return key material" |
| ONBOARDING_SPEC.md:236-241 | yes | "OWNER-CALLs recorded by the init panel, not silently resolved ... Defaults ship as: 50/day, viewports unchanged, 3 devices" |
| ONBOARDING_SPEC.md:245-249 | yes | inventory: "POST /auth/device/approve (session-authed, kind-aware)"; "capture validation for trial options"; no render path named |
| 0001_initial_schema.sql:9,16-18 | yes | `email TEXT NOT NULL UNIQUE`; idx_accounts_email; partial idx_accounts_stripe |
| 0001_initial_schema.sql:32, :57 | yes | FOREIGN KEY (account_id) REFERENCES accounts(id) on api_keys and items |
| 0002_auth_sessions.sql:5-13 | yes | magic_tokens columns id,email,token_hash,created_at,expires_at,used_at,ip_hash - no purpose |
| 0002_auth_sessions.sql:21,29 | yes | sessions.account_id TEXT NOT NULL, FK accounts(id) |
| 0006_mcp_devices.sql:15-16 | yes | FKs to accounts and api_keys |
| 0009_api_keys_override.sql:2 | yes | `ALTER TABLE accounts ADD COLUMN max_api_keys_override` |
| cleanup.ts:14-24 | yes | SELECT expired items ... `if (!expired.results || expired.results.length === 0) { return; }` |
| cleanup.ts:188-191 | yes | items tombstoned `status = 'deleted'` |
| auth-verify.ts:38-77 | yes | upserts account by email, inserts session bound to account.id, 302 /dashboard with wn_session |
| api-keys.ts:136-143 | yes | revoke key; cascade revoke mcp_devices; no account deletion |
| mcp.ts:291-293 | yes | `if (!auth) return jsonRpcError(null, -32000, ...)` before body handling |
| mcp.ts:335 | yes | enrollment gated `auth.type === "api_key" && isInitializeRequest(body)` |
| env.ts:38-41 | yes | feature flags: ZIP_BUNDLES, EMAIL_SHARING, FACE_BLOCKING; no ANON_TRIAL |
| router.ts:154-155 | yes | `POST /api/v1/render` -> handleRender |
| render.ts:33-35, 56-66, 118-122 | yes | auth required; up to 6 viewports, full_page; allocates R2 before any quota check in the shown span |
| auth.ts:174-192 | yes (by hand) | active key + active account accepted; plan returned, not filtered |
| account-items.ts:19-20 | yes | history scoped `account_id = ?` |
| capture.ts:233-237 | yes | daily upload limit check (checkDailyUploadLimit) |
| capture.ts:494, 658-674, 712 | yes | note ?? `Google Doc: ${docTitle}`; note ?? defaultNote (`Screenshot of ${url}`); `captured_url: url` in the JSON response only |
| rate-limit.ts:20-21, :70 | yes | no CF-Connecting-IP -> allowed; PLAN_LIMITS[plan] ?? free |
| types.ts:32 | yes | AccountPlan = free|pro|team|owner (no trial) |

Verified-by-inspection facts (used for `VERIFIED` provenance in the ledger):
V1 email is NOT NULL UNIQUE and every later migration is ADD COLUMN; the spec's migration text is one clause (A2, B2).
V2 sessions.account_id is NOT NULL and both verify paths create no account; approve is specified session-authed (A4, B4).
V3 trial_counters has an UPDATE and no INSERT/UPSERT in the spec (A7, B3).
V4 cleanup.ts returns early when no items expired; every later task sits after the return (A3 second half, B5).
V5 api_keys, items, mcp_devices, sessions all carry FKs to accounts; the spec's cleanup names "delete the account row" with no dependent handling (A3 first half; whether D1 enforces FKs is EXTERNAL).
V6 the spec names a response header for the raw key without naming it, and the tree has no anon-trial feature flag or wrapper discriminator (B6; A1's "raw key" clause).
V7 the spec labels quota 10 and Turnstile as OWNER-CALL, ships "3 devices" as a default, and records raw-key vs token as UNRESOLVED while the issuance section delivers a raw key (A1, B1: the labelled-vs-silent distinction is a matter of reading, recorded as contested).
V8 magic_tokens has no purpose column; the spec's per-IP cap names no table (B3; A2's purpose clause).
V9 device_codes sketch has no UNIQUE partial index; note is client-replaceable and captured_url is response-only (B8; A5's "no key/device identifiers").
V10 the render route accepts any active key with no plan check and is absent from the spec's enforcement inventory (A8).
V11 auth.ts accepts an active key regardless of plan; account-items scopes by account (A6 mechanism facts; the attacker walkthrough itself is a PANELIST-CLAIM).
