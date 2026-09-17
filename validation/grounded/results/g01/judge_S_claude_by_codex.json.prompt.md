You are scoring one review memo against a sealed rubric. You are not a participant; do not add your own review.
Score ONLY from the memo text. Quote the memo span that earns each point; if no span supports a point, the score is 0.

=== SEALED RUBRIC ===
# G01 sealed rubric - SEALED (arms never see this)

- **difficulty:** design (no single correct call; scored on must_catch and landmines)
- **correct_call:** A thin browser **device-code authorization** layer wrapped
  around the existing magic-link flow: `whipnode init` (also triggered by the
  MCP `-32000` unauthenticated error) starts a device code; the human proves
  the mailbox via the existing magic link; the single consent moment is one
  "Authorize this device" click on a page that names the device and the
  free-plan limits; the worker mints **at most one** backing `wn_` key per
  free account and enrolls a per-installation `mcp_devices` row; the CLI
  stores the credential in a 0600 file; a smoke capture ends the flow; the
  recipe page lives on whipnode.com. Anonymous/trial keys and unattended
  signup are out (under this brief's consent constraint).
- **oracle:** the architecture was built and shipped exactly so (repo commit
  57e9eff and the released `npx whipnode` 0.1.0; `outcome_source` below).
  Artifact facts that force it: `apps/worker/src/routes/auth-verify.ts:38-50`
  (account is created only on magic-link consume, `plan='free'`) and
  `:71-77` (verify always 302s to `/dashboard`, no `next`);
  `apps/worker/src/routes/mcp.ts:291-293` (MCP refuses unauthenticated
  calls with -32000) and `:333-387` (device enrollment on `initialize`,
  `Mcp-Session-Id` header, quota 10 per key);
  `apps/worker/migrations/0006_mcp_devices.sql:1-17`;
  `packages/protocol/src/constants.ts:42-52` (free = 50/day, 24 h, 1 API key);
  `apps/worker/src/middleware/auth.ts:114-119` (90-day sliding device
  session); `apps/worker/src/routes/api-keys.ts:134-147` (revoke cascades to
  devices).
- **correct fix / build list:** a `device_codes` table; `POST /auth/device/start`,
  `poll`, `approve`; a `/device` approve page; a `next` parameter on the
  magic-link verify (today it hard-redirects to `/dashboard`); an
  `npx whipnode` CLI (none exists: `docs/ROADMAP.md:36` lists "CLI tool for
  terminal workflows" as future); unauth-MCP error copy pointing at init.
- **must_catch:**
  1. Free accounts get exactly one API key (`constants.ts:49`
     `max_api_keys: 1`); per-device segregation already exists as
     `mcp_devices` rows under that key (`0006_mcp_devices.sql`, `mcp.ts:333-352`),
     so init must not mint per-device keys.
  2. Accounts are created only on magic-link consume (`auth-verify.ts:38-50`);
     there is no device/authorization-code exchange in the code today, and
     verify hard-redirects to `/dashboard` (`auth-verify.ts:71-77`), so the
     flow needs a `next`/return path.
  3. There is no CLI today (`docs/ROADMAP.md:36`); the current agent path is
     magic link -> dashboard -> create key -> paste `claude mcp add ... X-API-Key`
     (`apps/web/app/components/ApiKeyManager.tsx:70`).
  4. Email verification alone is a weak farming gate: 3 links/email/15 min
     (`auth-magic-link.ts:23-33`), no Turnstile anywhere in the worker or web
     app (grep: none), captures launch Puppeteer and are the expensive
     resource (`capture.ts` imports puppeteer; up to 6 viewports per request,
     `capture.ts:546-548`).
  5. The consent moment is the browser Authorize click, not the email-link
     click (mailbox proof is identity, not authorization).
  6. Raw keys must be delivered once, to the verifier-bound CLI, and never
     re-displayed (keys are stored hashed; `mcp.ts:333-367` pattern).
- **landmine (confident wrong answers):** "issue an anonymous trial key on first
  MCP call" (violates this brief's consent constraint); "mint a fresh API key
  per device" (free tier allows one); "reuse magic-link as-is, the email
  click is the consent" (verify creates the account and redirects to the
  dashboard with no return path); "Turnstile already protects login" (it does
  not exist in the tree).
- **unverifiable:** the three parameter choices the owner later made (raw key
  vs device token to the CLI; Turnstile day-one; 50/day after claim with a
  25/day anonymous trial) are owner calls, not derivable from the artifact -
  do not score them.
- **outcome_source:** `2026-08-19-whipnode-init-design/retro.md` (T0 agreed_r0
  list; T1 outcome "Built per the unanimous architecture ... PR #3 / npm
  0.1.0"; verdict_held yes); seat outputs `codex-r0.md` claims 1-4,
  `grok-r0.md` C1-C4 and FLOW SPEC.


=== MEMO (author unknown) ===
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


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}