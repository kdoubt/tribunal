## Framing

Three plausible framings: **(A)** greenfield device-authorization (OAuth 8628) flow; **(B)** thin CLI orchestrating the *existing* magic-link → api-key → MCP-device-enrollment chain, adding one new primitive; **(C)** anonymous trial key with claim-later upgrade. I pick **(B)**. (C) is ruled out by `capture.ts:229` — captures hard-require an `auth.accountId`, so an anonymous trial would need a phantom account row, i.e. unattended account creation. (A) over-builds: the device half already exists (Claim 1). (B) matches the "reuse existing tables" criterion with exactly one new table.

## Claims

```
CLAIM: MCP device enrollment is already automatic — an API key presented on `initialize` mints a 90-day `Mcp-Session-Id` and an mcp_devices row, so `whipnode init` needs no new device-auth mechanism, only a way to obtain the first key.
EVIDENCE: apps/worker/src/routes/mcp.ts:335 `if (auth.type === "api_key" && isInitializeRequest(body))` … :386 `headers.set("Mcp-Session-Id", sessionToken);`
CONFIDENCE: 0.93
FALSIFIER: A client that sends X-API-Key on initialize and receives no Mcp-Session-Id header.
```
```
CLAIM: The only genuinely new server component is an init-code table in D1, because magic-link verification's sole terminal action is a 302 + cookie to the dashboard, and there is no KV binding to hold a pending code.
EVIDENCE: apps/worker/src/routes/auth-verify.ts:72-77 (`Location: ${env.APP_BASE_URL}/dashboard`, `Set-Cookie: wn_session=…`); apps/worker/src/env.ts:3-7 lists only DB/R2/QUEUE/BROWSER/AI.
CONFIDENCE: 0.88
FALSIFIER: An existing route or table that returns a key to a non-browser poller.
```
```
CLAIM: Magic-link rate limiting is per-email only, so today's signup surface offers no resistance to one-account-per-disposable-inbox farming, even though ip_hash is captured.
EVIDENCE: apps/worker/src/routes/auth-magic-link.ts:26 `WHERE email = ? AND created_at > ?`; :47 binds `ipHash` but no query reads it.
CONFIDENCE: 0.9
FALSIFIER: An IP- or domain-scoped counter elsewhere on the magic-link path.
```
```
CLAIM: There is no bot challenge (Turnstile/captcha) anywhere in the tree, so adding one to the init path is net-new work, not configuration.
EVIDENCE: repo-wide grep for `turnstile|captcha|hcaptcha|recaptcha` (case-insensitive) → "No matches found"; apps/web/app/routes/login.tsx:72-89 is a bare email form.
CONFIDENCE: 0.92
FALSIFIER: A Turnstile sitekey in wrangler.toml or a WAF rule outside the tree (SPECULATIVE: Cloudflare dashboard config is not readable here).
```
```
CLAIM: Free accounts cannot have per-device API keys; segregation must be per-device rows under a single key (free = 1 key × ≤10 devices).
EVIDENCE: packages/protocol/src/constants.ts:48 `max_api_keys: 1`; apps/worker/src/routes/mcp.ts:352 `WHERE (SELECT COUNT(*) FROM mcp_devices WHERE api_key_id = ? AND status = 'active') < 10`.
CONFIDENCE: 0.9
FALSIFIER: handleCreateApiKey issuing a 2nd active key on a free plan (api-keys.ts:42 blocks it).
```
```
CLAIM: The abuse-cost asymmetry sits on captures, not uploads — each capture spawns a Puppeteer browser plus R2 writes plus a queue/OCR job, yet free is metered only by the shared 50 uploads/day counter.
EVIDENCE: apps/worker/src/routes/capture.ts:565 `puppeteer.launch(env.BROWSER)` inside the per-viewport loop; :234 `checkDailyUploadLimit`; constants.ts:44 `uploads_per_day: 50`.
CONFIDENCE: 0.85
FALSIFIER: A separate capture-specific quota in the enforcement path.
```
```
CLAIM: The pricing page already misstates Pro limits relative to code, so the upsell copy is dishonest-by-staleness before any funnel is built.
EVIDENCE: apps/web/app/routes/pricing.tsx:47 `<li>50 MB per file, 100 MB per item</li>` vs packages/protocol/src/constants.ts:55 `max_total_bytes_per_item: 250 * 1024 * 1024`.
CONFIDENCE: 0.87
FALSIFIER: An override making pro items 100 MB at runtime.
```
```
CLAIM: A no-dashboard upsell surface already has a home — whipnode_status prints plan and today's count but never prints the remaining budget or the wall being approached.
EVIDENCE: apps/worker/src/routes/mcp.ts:673-674 `Account: … (${account?.plan ?? "free"} plan)` / `Uploads today: ${uploadCount?.cnt ?? 0}` — no limit rendered; compare rate-limit.ts:85 which only speaks at refusal time.
CONFIDENCE: 0.86
FALSIFIER: Any MCP response containing PLAN_LIMITS values pre-refusal.
```

## FLOW SPEC

1. User reads the whipnode.com recipe page; copies `npx whipnode init`.
2. CLI POSTs `/api/v1/init/start` (new) with a client-generated code_verifier; server inserts a row into a new `init_codes` table (D1; mirror `magic_tokens` shape — `token_hash`, `expires_at`, `used_at`, `ip_hash`), 10-min TTL.
3. CLI prints a URL `…/init?code=XXXX-XXXX` and the user-code, then long-polls `/api/v1/init/poll`.
4. **Consent moment (the one human act):** user opens that URL in a browser, sees device name + scope + "create a free WhipNode account for `<email>`", types email, passes Turnstile, clicks **Approve**. Nothing before this creates an account.
5. Browser POSTs `/api/v1/auth/magic-link` with the init code attached (reuses `auth-magic-link.ts` verbatim plus one column).
6. User clicks the emailed link → `auth-verify.ts` runs unchanged, upserting the account (`auth-verify.ts:43-51`) and setting the session cookie.
7. New branch in the verify handler: if the token carries an init code, bind `init_codes.account_id` and redirect to `/init/done` instead of `/dashboard`.
8. `/init/done` (or the verify branch itself) calls `handleCreateApiKey` with `name = "<device> via init"` — existing route, no change (`api-keys.ts:49-58`).
9. CLI's poll returns the raw key once; `init_codes.used_at` is stamped atomically with the same `UPDATE … WHERE used_at IS NULL` idiom as `auth-verify.ts:22`.
10. CLI writes the key to `~/.whipnode/credentials.json` (0600), **not** env or shell rc, and registers the MCP server with the `claude mcp add … --header "X-API-Key: …"` command already documented at `docs.tsx:43`.
11. First `initialize` enrolls the device and swaps the key for a session token (`mcp.ts:335-387`); CLI then rewrites the stored config to drop the raw key.
12. CLI issues one `whipnode_capture` of the recipe page as a smoke test and prints the share URL. Failure/retry: poll timeout → re-run prints the same URL for 10 min; expired code → fresh code, no account touched; email never arrives → `--paste-key` fallback to the dashboard path.

## TIER TABLE

| Dimension | Free | Pro |
|---|---|---|
| Uploads/day | 50 (constants.ts:44) | 1000 (constants.ts:54) |
| **Captures/day** (new sub-quota) | 15 | 300 |
| Viewports per capture | 2 | 6 (capture.ts:546 cap) |
| Retention | 24 h (`ttl_hours_max: 24`) | 168 h |
| Per-file / per-item | 25 MiB / 25 MiB | 50 MiB / 250 MiB |
| History (`whipnode_list`) | last 24 h only (`history: false`) | full (`history: true`) |
| API keys × devices | 1 × 10 | 10 × 10 |
| OCR | included | included, priority queue |

## UPSELL TRIGGERS

1. Capture sub-quota hit — refusal text names the remaining-vs-limit number, not a discount.
2. Item expires at 24 h while an agent is still polling `whipnode_list` for it.
3. `viewports: [desktop, tablet, mobile]` requested on free → runs the first two, states the third needs Pro.
4. Second device enrolls and the user asks for group isolation (`mcp.ts:701`) — grouping is only meaningful above 1 key.
5. Upload rejected at 25 MiB with the honest byte count.

## VERDICT INPUT

- **Q1:** Reuse magic-link end to end; add exactly one primitive (`init_codes` in D1) plus a browser consent screen; device auth, key issuance, and revocation are already built (`mcp.ts:335`, `api-keys.ts`, `mcp-devices.ts`).
- **Q2:** Free tier needs a capture-specific sub-quota carved out of the 50/day counter; upsell lives in `whipnode_status`/refusal strings for CLI-only users — and `pricing.tsx:47` must be corrected before it is a funnel.
- **Q3:** Email verification alone is not enough — the per-email-only limiter (`auth-magic-link.ts:26`) and the total absence of a challenge mean Turnstile on the consent screen plus an IP-hash counter are prerequisites, not hardening.
