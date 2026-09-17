# Oracles — g02 (orchestrator-run, by inspection only; no arguments added)

Pointer resolution: tools/grounding.py resolved 16/16 of Seat B's pointers. It found 0 pointers for Seat A because Seat A writes
`**EVIDENCE:**` (bold) and the tool matches `^EVIDENCE:` only - a tool limitation, logged; Seat A's pointers were opened by hand
below and all 21 resolve. (grounding_rate for A must be recomputed with a bold-tolerant pattern at scoring time.)

Opened spans and what they say (artifact = whipnodeprivate tree; W = apps/worker/src, P = packages/protocol/src):
- W/routes/capture.ts:25 `const MAX_WAIT_MS = 25_000;` | :546 `if (viewportNames.length > 6)` | :567-621 loop: `browser.newPage()` per viewport, goto with timeout MAX_WAIT_MS, `setTimeout(r, 3000)` at :579, screenshot `{type:"png", fullPage}` at :606, `await browser.close()` at :621 inside try; catch at :622 returns apiError without closing the browser (no `finally` in the file) | :20 `"4k": {width:3840,height:2160}` | :699 filter `.endsWith(".png")` then QUEUE.send.
- W/routes/capture.ts:234 `checkDailyUploadLimit(auth.accountId, auth.plan, env)`; grep: `checkUploadRateLimit` is called only at W/routes/items-create.ts:70. → A2 VERIFIED.
- W/routes/capture.ts:653-680: `ip = request.headers.get("CF-Connecting-IP")`, `ipHash`/`uaHash` stored in `items` (source_ip_hash, user_agent_hash). packages/shared/src/hash.ts:20-25: `sha256Hex("ip:"+ip)` deterministic. → B4 VERIFIED.
- W/middleware/rate-limit.ts:21 `if (!ip) return { allowed: true };` | :70 `const limits = PLAN_LIMITS[plan] ?? PLAN_LIMITS.free;` | :74-78 `SELECT COUNT(*) ... FROM items WHERE account_id = ? AND created_at > ?`. → A3 (with mcp.ts:953), A6, B2-count clause VERIFIED.
- P/constants.ts:43-51 free: uploads_per_day 50, ttl_hours_max 24, max_api_keys 1, history false | :105-117 ABUSE_BUDGET six constants. grep: MAX_OCR_RUNS_PER_IP_PER_DAY, MAX_BANDWIDTH_PER_IP_PER_HOUR, MAX_DOWNLOADS_PER_ITEM_PER_HOUR appear only in constants.ts. → A4, B3-evidence, B6 VERIFIED.
- docs/private/ABUSE_PREVENTION.md:18-20 lists those limits with enforcement points "rate limiting middleware" / "queue consumer". → A4 doc clause VERIFIED (path is docs/private/, not docs/).
- apps/worker/migrations/0001_initial_schema.sql:9 `email TEXT NOT NULL UNIQUE`; no migration 0002-0012 alters accounts.email (grep). → A5 constraint, B4 email clause VERIFIED.
- W/routes/mcp.ts:291-293 "Authentication required. Pass your API key in the X-API-Key header." | :319-320 `Promise.all(body.map(...handleSingleRequest...))` | :335 `if (auth.type === "api_key" && isInitializeRequest(body))` | :341 reads CF-Connecting-IP from the outer request | :386 `headers.set("Mcp-Session-Id", sessionToken)` | :948-953 internal Request with only Content-Type; comment "CF-Connecting-IP not forwarded" | :1009-1013 same for captures. → A3, A7, B2 clauses VERIFIED.
- W/routes/auth-verify.ts:21-23 atomic `UPDATE magic_tokens SET used_at ... WHERE ... used_at IS NULL AND expires_at > ?` | :39-51 select-or-create account by email. → A8 single-use clause, B5 pointer VERIFIED.
- W/scheduled/cleanup.ts:7-8 "Finds expired items, deletes R2 objects, marks items as deleted." → B6 VERIFIED.
- apps/web/app/routes/index.tsx:181-184 install command with X-API-Key header; "Create an account to get your API key." → B7 pointer VERIFIED.

Not oracle-settleable (design/threat claims, left PANELIST-CLAIM/ASSUMPTION): A5 sentinel-vs-nullable, A7 vs B ladder mechanism,
A8 vs B5 (which binding proof defeats attacker-authored links), B3 economics, B ladder 11 reissue vs A ladder 11 same-key.

## Addendum after Round 1 (new pointer cited by Seat B)
- W/middleware/auth.ts:30-33: `Mcp-Session-Id` header → `resolveMcpDevice(mcpSessionId, env)`; :72-83: device row resolved by `session_token_hash` with JOINs to `api_keys` and `accounts`; :103-104: `if (row.key_status !== "active") return { type: "rejected", reason: "key_revoked" }`. All three spans open and say what B's R1 states (a presented session token authenticates its holder; a device's parent key status gates it). By inspection only; no support rating.
