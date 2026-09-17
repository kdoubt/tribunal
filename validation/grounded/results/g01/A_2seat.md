A_2seat = union of two isolated Round 0 memos, no ledger, no Round 1

=== Memo 1 ===

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


=== Memo 2 ===

I'll ground this in the WhipNode checkout first: read the auth, MCP, capture, and pricing surfaces, then take an independent position on the init flow.Next I’ll read the required auth, MCP, capture, and pricing files so claims stay tied to the actual code.I’ll keep reading capture, migrations, keys, and the docs that define current limits and onboarding.**Framings.** Q1 alternatives: (1) RFC 8628-style device-code over existing magic-link + `api_keys` + `mcp_devices`; (2) magic-link-only CLI poll with no browser confirm; (3) anonymous trial key, claim later; (4) keep dashboard copy-paste; (5) new OAuth/OIDC. **Pick (1):** one human browser act, reuses D1 tables, rejects unattended create, fits ~2 minutes. Q2: (1) generous uploads/OCR, honest walls on capture/viewports/TTL/keys; (2) starve free for revenue; (3) keep “early access, all features.” **Pick (1).** Q3: (1) Turnstile on consent + email + device cap + capture cost; (2) email-only; (3) SMS. **Pick (1).**

---

CLAIM: MCP and docs today cannot take an agent from nothing to a capture: unauthenticated MCP returns JSON-RPC `-32000` asking for `X-API-Key`, and Getting Started is login → create key → paste `claude mcp add … --header "X-API-Key: …"`.
EVIDENCE: `apps/worker/src/routes/mcp.ts:291-292` `"Authentication required. Pass your API key in the X-API-Key header."`; `apps/web/app/routes/docs.tsx:29-43`; `apps/worker/src/routes/capture.ts:229-231` `"URL captures require an API key."`; ROADMAP “CLI tool” is Future (`docs/ROADMAP.md:36`).
CONFIDENCE: 0.95
FALSIFIER: An unauthenticated `POST /api/v1/mcp` `tools/call` `whipnode_capture` returns 201, or a `whipnode` CLI binary exists in-tree.

CLAIM: Do not issue anonymous trial keys or auto-create accounts; the single consent moment is a human click on `/init` “Authorize this device” after a session cookie or magic-link verify, which upserts `accounts.plan='free'` and enrolls the device.
EVIDENCE: Account insert only in `apps/worker/src/routes/auth-verify.ts:43-50` (`plan, 'free'`); email is “Sign in to WhipNode”, not device consent (`apps/worker/src/lib/email.ts:23-32`); USER-FACT “unattended account creation without human consent is unacceptable.”
CONFIDENCE: 0.9
FALSIFIER: Shipping an API that inserts `accounts` or `api_keys` with no browser/email human act.

CLAIM: Build a short-lived device-code pairing API + `/init` page (new); reuse magic-link (15 min, 3/email/15 min), one `api_keys` row, and `mcp_devices` insert/session (90-day sliding, max 10 active per key) instead of OAuth.
EVIDENCE: `apps/worker/src/routes/auth-magic-link.ts:23-32,39`; `apps/worker/src/routes/mcp.ts:334-367,344`; `apps/worker/src/middleware/auth.ts:114-115`; `apps/worker/migrations/0006_mcp_devices.sql`; grep has no device-code/Turnstile/OAuth. EXTERNAL: RFC 8628.
CONFIDENCE: 0.78
FALSIFIER: Pairing implemented as a new identity provider, or as email-click-only with no device row.

CLAIM: Free-tier segregation is per-device `mcp_devices` sessions, not per-device API keys: `PLAN_LIMITS.free.max_api_keys` is 1 and create-key 403s at the cap; pairing must reuse/create that one `["write"]` key and return a device session (raw keys are shown once and only hashed thereafter).
EVIDENCE: `packages/protocol/src/constants.ts:42-51`; `apps/worker/src/routes/api-keys.ts:42-46,53-66`; `apps/web/app/components/ApiKeyManager.tsx:309-310`; `apps/worker/src/middleware/auth.ts:26-52` (`Mcp-Session-Id` then `X-API-Key`). ASSUMPTION: Claude Code persists a `Mcp-Session-Id` header if `whipnode init` writes it.
CONFIDENCE: 0.82
FALSIFIER: Free accounts already mint many active keys, or pairing returns a second raw key without rotating/reusing.

CLAIM: Shape free for this funnel as: 50 uploads/day, OCR, 25 MB, 24 h TTL, 1 API key, 1 viewport per capture, ~50 captures/month; Pro walls are extra viewports/4k, 50 MB/file, 7 d TTL, history, 10 keys — viewports are not plan-gated today (`max 6` only).
EVIDENCE: `packages/protocol/src/constants.ts:42-62`; `apps/web/app/routes/pricing.tsx:27-53`; `apps/worker/src/routes/capture.ts:536-548`; USER-FACT 50 MB paid gate. Stale: `apps/web/app/routes/terms.tsx:61` “No API access” vs `login.tsx:94-95` “1 API key” and `PLAN_LIMITS.free.api_access: true`.
CONFIDENCE: 0.8
FALSIFIER: `handleCapture` already 403s `4k`/multi-viewport on `plan==='free'`, or Browser Rendering is billed such that 50 captures/day is cheaper than email farming.

CLAIM: For users who never open the dashboard, the upsell is the existing limit error that names the plan plus `whipnode_status`’s `Account: … (plan)` line — one sentence, only on the wall; the methodology recipe lives on whipnode.com (init URL, what Authorize grants, one capture example, free limits, `/pricing`), not in the unnamed repo.
EVIDENCE: `apps/worker/src/middleware/rate-limit.ts:82-85`; `apps/worker/src/routes/mcp.ts:672-673`; USER-FACT recipe-on-whipnode.com; `apps/web/app/routes/pricing.tsx:43-45` Pro price TBD.
CONFIDENCE: 0.86
FALSIFIER: A capture-limit response with no plan/upgrade pointer, or the methodology repo naming WhipNode.

CLAIM: Email verify is necessary and not sufficient against farming: magic-link has no CAPTCHA, only `COUNT(*) … >= 3` per email; add Turnstile on `/init`, cap free active devices to 3 (today 10/key), 10-minute pairing TTL, keep hashed IP; the agent may start/poll pairing and capture after, and must not read mail or click the link.
EVIDENCE: `apps/worker/src/routes/auth-magic-link.ts:23-32,40-47`; `apps/worker/src/routes/mcp.ts:347-352`; `docs/private/ABUSE_PREVENTION.md` (IP hash, no Turnstile); `mcp.ts:214-217` configure/apply already requires human confirm.
CONFIDENCE: 0.77
FALSIFIER: Disposable-email farms creating many free accounts through `/init` with Turnstile+3-device cap in place at similar cost to one capture.

CLAIM: Init should store only email, API-key hash, device_name, ip_hash, user_agent_hash, and 24 h items; revocation/expiry already exist (device revoke, key cascade-revoke, 90-day inactive, sliding `expires_at`).
EVIDENCE: `docs/PRIVACY.md:3-16`; `apps/worker/src/routes/mcp-devices.ts:70-74`; `apps/worker/src/routes/api-keys.ts:139-143`; `apps/worker/src/routes/mcp.ts:282-284`; `apps/web/app/components/McpDeviceManager.tsx:77`.
CONFIDENCE: 0.88
FALSIFIER: Init persists raw files, OCR, or unhashed IPs beyond those columns, or revoke leaves the session valid on the next MCP call.

### FLOW SPEC
1. User has no account/key. Agent or `npx whipnode init` (new; MCP-first: unauth tools return pairing instead of only `mcp.ts:292`).
2. `POST /api/v1/device-codes` → user_code + verification_url `https://whipnode.com/init?code=…` (TTL ~10 min). Agent prints the URL; does not open mail.
3. Human opens `/init`. If no `wn_session`, email form → existing magic-link (`auth-magic-link.ts`); verify redirects to `/init?code=` not `/dashboard` (`auth-verify.ts:73`).
4. Consent page names device (UA parse as `parseDeviceName`) and grants: free account, one write key, this MCP/CLI device may upload/capture/list; Turnstile.
5. Human clicks **Authorize** (the one consent act). Server: upsert account if needed; reuse or create the single `api_keys` row; `INSERT mcp_devices` (same shape as `mcp.ts:348-352`) with 90-day expiry.
6. CLI/MCP poll succeeds once; response is device session token (and raw key only if newly created). Write MCP header (`Mcp-Session-Id` or `X-API-Key`) and `WHIPNODE_API_KEY` if issued.
7. Dashboard copy-paste remains a fallback (`ApiKeyManager` Claude/Codex blocks) for humans who already have a key.
8. Retry: expired code → new code; magic-link 3/15 min → wait; device cap → “revoke one at /dashboard#devices”; revoked/expired device → existing `mcp.ts:282-284` copy pointing at re-init, not a silent drop (`mcp.ts:369-375` quota skip is the bad pattern to avoid on init).
9. First `whipnode_capture` with the new session (`capture.ts` + daily limit).
10. Tool result includes share/prompt URLs (`mcp.ts:1046-1053`).
11. `whipnode_status` shows `email (free plan)` and this device.
12. On 24 h TTL or capture/viewport wall, one upgrade sentence + `whipnode.com/pricing`.

### TIER TABLE

| Dimension | Free (init default) | Pro |
|---|---|---|
| Uploads/day | 50 (existing) | 1000 |
| Captures | 50/month (new meter; Browser Rendering) | 1000/month |
| Viewports | 1 (desktop); no 4k | up to 6 incl. 4k |
| File / item | 25 MB | 50 MB/file (existing paid gate) |
| Retention | 24 h | 168 h |
| OCR | yes | yes |
| API keys / devices | 1 key; 3 active devices | 10 keys; 10 devices/key |
| History | `history: false` (enforce on `account-items` / `whipnode_list` beyond 24 h) | yes |
| Rate | 60 MCP/min, 20 uploads/IP/h (existing) | same + priority |

### UPSELL TRIGGERS
1. Capture monthly cap hit → tool error names free vs Pro, link `/pricing`.
2. Second viewport / `4k` requested → “multi-viewport is Pro”; still return the desktop shot.
3. `ttl_hours > 24` or fetch after expiry → 410 + extend-on-Pro.
4. File > 25 MB → existing paid-upload copy (`items-create.ts:61-65`).
5. 4th device or 2nd API key → revoke or upgrade; no dark pattern, no recurring nag.

### VERDICT INPUT
Q1: Device-code `/init` Authorize over magic-link + one `api_keys` + `mcp_devices` session; no anonymous keys; dashboard paste stays fallback.  
Q2: Adoption-first free (50 uploads/day, OCR, 1 viewport, 50 captures/month, 24 h); Pro = viewports, 50 MB, 7 d, keys/history; upsell in MCP errors + whipnode.com recipe.  
Q3: Consent is the Authorize click; Turnstile + email + 3-device cap + capture cost; agent polls only; store email/hashes/24 h items; reuse revoke/90-day expiry.
