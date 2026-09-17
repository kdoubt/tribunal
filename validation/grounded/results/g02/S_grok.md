I'll read the WhipNode artifact first and ground claims in the actual capture, auth, and rate-limit code before taking a position.I'll pull the capture path, rate limits, auth, MCP, and abuse docs next so claims can cite real gates rather than the proposal text.Next I’ll read capture, MCP, magic-link, and schema files so cost, issuance, and claim binding can be pinned to existing tables.**Framings considered:** (1) growth/onboarding — is zero-click the right first mile; (2) resurrect consent for PII-less accounts; (3) abuse economics of unauthenticated Chromium; (4) claim-link as a transcript-bearer; (5) ladder vs `whipnode init` as a product-surface question. I take **(3)+(4) as the veto pair**: the owner named abuse as the centre question, and the brief forbids any secret that works when pasted from a log. Consent for account *creation* stays dissolved (USER-FACT: no email, no subject). Linking email later is the existing Authorize click, not a new signup.

---

CLAIM: A single `whipnode_capture` can pin Cloudflare Browser Rendering for up to 25s per viewport (plus a 3s heavy-SPA settle) across as many as 6 viewports including 4k and `full_page`, and that cost is **not** bounded by `ABUSE_BUDGET` IP caps because `handleCapture` never calls `checkUploadRateLimit`.
EVIDENCE: `capture.ts:25` `MAX_WAIT_MS = 25_000`; `capture.ts:14-21` VIEWPORTS including `"4k": { width: 3840, height: 2160 }`; `capture.ts:546-548` “Maximum 6 viewports”; `capture.ts:576-578` SPA `setTimeout(..., 3000)`; `capture.ts:233-237` daily account limit only; `rate-limit.ts:16-40` IP hourly cap is `checkUploadRateLimit`, imported by `items-create.ts:70` not `capture.ts`. MCP then strips the client IP: `mcp.ts:1009-1013` builds `new Request("https://internal/api/v1/captures")` with no `CF-Connecting-IP`.
CONFIDENCE: 0.92
FALSIFIER: `handleCapture` invoking `checkUploadRateLimit` on a request that still carries `CF-Connecting-IP`, or a viewport/timeout cap below these constants.

CLAIM: Naked auto-provision on first unauthenticated MCP call is not survivable on the current schema or gates: MCP rejects missing auth, `accounts.email` is `NOT NULL UNIQUE`, `AccountPlan` has no trial, and nothing in the worker reads `request.cf` bot score or ASN.
EVIDENCE: `mcp.ts:291-293` `if (!auth) return jsonRpcError(..., "Authentication required. Pass your API key...")`; `0001_initial_schema.sql:9` `email TEXT NOT NULL UNIQUE`; `types.ts:32` `AccountPlan = "free" | "pro" | "team" | "owner"`; repo grep of `request.cf` / `botScore` / `ASN` matches only the brief.
CONFIDENCE: 0.9
FALSIFIER: A migration making `accounts.email` nullable plus an existing unauthenticated MCP initialize path that already inserts an account.

CLAIM: Rung-0 issuance must create `accounts` (nullable email, `plan='trial'`) + `api_keys` + `mcp_devices` in one D1 batch on `initialize`, return **only** `Mcp-Session-Id` (existing enrollment header), and refuse unless `FEATURE_ENABLE_TRIAL_ISSUANCE` is on, this IP hash has minted fewer than 1 trial in 24h, and a global daily trial-capture counter is under budget.
EVIDENCE: Enrollment already sets the header and never echoes the raw key: `mcp.ts:335-387` `headers.set("Mcp-Session-Id", sessionToken)` after hashed insert; feature-flag pattern: `env.ts:39-41` `FEATURE_ENABLE_*`; IP hash: `rate-limit.ts:20-23` `CF-Connecting-IP` → `hashIp`; missing IP currently fail-opens: `rate-limit.ts:21` `if (!ip) return { allowed: true }`. Printing `wn_` in tool text would violate the transcript-bearer constraint (USER-FACT + brief Constraints).
CONFIDENCE: 0.78
FALSIFIER: Observing a `wn_` key or claim secret in a JSON-RPC `tools/call` body on the shipped trial path, or trial mint succeeding with the feature flag unset.

CLAIM: The survivable trial envelope is 10 **browser jobs** (capture+render) per UTC day, forced single `desktop` viewport, `full_page` and `4k` rejected, TTL 24h (`DEFAULT_TTL_HOURS`), with file `whipnode_upload` sharing that 10; unconstrained 6×4k captures are not insurable for a small Worker+bindings bill.
EVIDENCE: Proposal “about 10 captures/day, 24h”; `constants.ts:38` `DEFAULT_TTL_HOURS = 24`; `constants.ts:43-51` free is already 50/day — trial must be a new `PLAN_LIMITS.trial` row, not `free`; `render.ts:18` `MAX_WAIT_MS = 15_000` and `render.ts:56` default three viewports. EXTERNAL: Cloudflare Browser Rendering (Workers Bindings) is 10 included hours/month then $0.09/browser-hour and 10 included concurrent browsers then $2/browser averaged (developers.cloudflare.com/browser-rendering/pricing/). Worst case in code: 6×(25s+3s) ≈ 2.8 min/job; a farm at 10 jobs × many minted trials exhausts included hours and the 10-concurrency grant.
CONFIDENCE: 0.7
FALSIFIER: Measured median browser-session duration under 2s with 4k/full_page disabled still, or a documented included-concurrency grant large enough that $2/browser is irrelevant at the global trial budget.

CLAIM: A claim URL that attaches email on GET (the current magic-link verify shape) is a transcript bearer and is unacceptable; the surfaced value may only be a public index `https://whipnode.com/claim` plus a short `WHIP-XXXX` user_code (15 min TTL, same as `magic_tokens`).
EVIDENCE: `auth-verify.ts:10-26` token in query string, atomic consume, then account upsert; `auth-magic-link.ts:39` 15 min expiry; `auth-magic-link.ts:51` `verifyUrl = .../auth/verify?token=`; brief Constraints: “Anything that becomes a bearer credential when pasted into a transcript is unacceptable.”
CONFIDENCE: 0.88
FALSIFIER: A shipped claim URL that, when opened in a logged-out browser with no `Mcp-Session-Id`, attaches an email or yields a session cookie for that trial.

CLAIM: Hijack (a) (leaked index claims the victim’s trial) and (b) (victim’s email bound to an attacker-held key) are both defeated only by a four-factor finish: bound `mcp_devices.id`, magic-link proof of email, Authorize page showing device name + last item notes/URLs, and a **browser-only** nonce that the bound device must POST; if `accounts.email` already exists, refuse in-place attach (do not merge).
EVIDENCE: Device binding exists: `mcp.ts:348-367` insert keyed by `api_key_id`, `auth.ts:74-81` session → `device_id`. Magic link + 3/15min throttle: `auth-magic-link.ts:23-32`. Verify currently upserts by email and would login the victim’s *existing* account, not the trial: `auth-verify.ts:38-50`. `mcp.ts:282-286` already returns device-revoked copy in-band — the claim nonce must **not** follow that pattern of putting a secret in the error. USER-FACT: prior init is device-code wrapping magic link + “Authorize this device”.
CONFIDENCE: 0.72
FALSIFIER: Completing a claim with only the user_code from a transcript, or attaching email E to trial T when E already has a row in `accounts`.

CLAIM: Rung 0 **complements** `whipnode init` (USER-FACT); it does not replace the consent path to a named free account. Claim reuses magic-link + the Authorize page, not CLI-secret device-code (that code would leak in the same transcript). Conversion is in-place: `UPDATE accounts SET email=?, plan='free'` keeping the same `api_keys` row and `mcp_devices` rows; do not rotate the key.
EVIDENCE: USER-FACT init design; current recipe always demands a key: `index.tsx:181` and `docs.tsx:43` `claude mcp add ... --header "X-API-Key: YOUR_KEY"`; `llms.txt:21` same; enrollment already ties devices to a key: `0006_mcp_devices.sql:3-7`. Revoke already cascades devices from the key: `api-keys.ts:139-143` — so keeping the key keeps devices.
CONFIDENCE: 0.74
FALSIFIER: Recipe still requiring `X-API-Key` after rung 0 ships, or conversion inserting a second `api_keys` row and revoking the trial key.

CLAIM: The kill switch is `FEATURE_ENABLE_TRIAL_ISSUANCE` off: unauthenticated `initialize` returns today’s auth error pointing at `whipnode init` / login; existing `plan='trial' AND email IS NULL` rows may read/fetch but `handleCapture`/`handleRender` reject; rows with email set and `plan` in `{free,pro,team,owner}` are untouched.
EVIDENCE: Flag slot: `env.ts:39-41`; capture/render already require auth: `capture.ts:229-231`, `render.ts:33-35`; MCP unauthenticated copy: `mcp.ts:292`. No current flag gates issuance, so this is additive, not a rename.
CONFIDENCE: 0.8
FALSIFIER: Disabling the flag blocking `plan='free'` captures, or unclaimed trial rows still launching `puppeteer.launch(env.BROWSER)`.

---

### LADDER SPEC
1. Recipe: `claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp` (no key). `whipnode init` remains for people who want a named account first.
2. First `initialize` with flag on: IP+global gates → insert trial account (email NULL), key (hash only), `mcp_devices` → `Mcp-Session-Id`. Target &lt;30s to first capture (issuance is D1, not email).
3. Tools run under `PLAN_LIMITS.trial`: 10 browser-jobs/day, 1×desktop, no 4k/full_page, 24h TTL.
4. Wall (daily cap, TTL, or flag off mid-session): JSON-RPC error with the copy below — user_code is an index, not a secret.
5. User opens `https://whipnode.com/claim`, types `WHIP-XXXX`. Page shows device_name, IP-prefix, last 3 item notes/URLs. Email field.
6. POST existing `/auth/magic-link` (15 min, 3/email). Mail copy says this will *claim a trial device*, not a generic sign-in.
7. Verify lands on Authorize (reuse init approve), **not** `auth-verify.ts`’s bare dashboard redirect.
8. If email already exists: stop with “This email already has a WhipNode account. Sign in there; this trial stays separate.” No merge.
9. Page displays a confirm nonce that never appeared in CLI/MCP output.
10. User pastes nonce into the same agent; bound device POSTs it. Other devices get `invalid confirm`.
11. `UPDATE accounts SET email=?, plan='free'`; same key, same `mcp_devices`; subsequent walls omit claim copy and name `/pricing`.
12. Flag off: step 2 stops; steps 11+ continue. Unclaimed trials cannot launch the browser.

**Error-response copy the agent relays**

> Trial limit reached (10 URL captures/day, 24h retention). This is not a sign-in link and will not work from a screenshot or someone else’s chat.
> Open https://whipnode.com/claim and enter code WHIP-XXXX from **this** session. Verify your email, then paste the confirm code shown **only on that page** back here.
> Do not use claim codes from email, Slack, or another terminal. Upgrade keeps this device and history (50/day). To skip the trial next time: whipnode init

### GUARDRAIL TABLE
| Threat | Control |
|---|---|
| Scripted mass mint | Flag + 1 trial / IP-hash / 24h; missing `CF-Connecting-IP` **denies** (invert `rate-limit.ts:21`); global daily trial-capture counter |
| Capture IP bypass via MCP fake Request | Copy `CF-Connecting-IP` into the internal Request; apply `checkUploadRateLimit` inside `handleCapture`/`handleRender` |
| 6×4k/full_page browser-hour farm | Trial plan rejects those args; 10 browser-jobs/day |
| Transcript paste claims account (a) | user_code is an index; finish needs bound `mcp_devices` + browser nonce |
| Phished email attach to attacker key (b) | Authorize shows attacker device + recent URLs; nonce lives on the page; no auto-confirm; refuse if email exists |
| `wn_` or session token in tool text | Header-only `Mcp-Session-Id`; never put raw keys in JSON-RPC |
| Kill switch hitting paying users | Gate **issuance** only; claimed rows keyed by `email IS NOT NULL` |
| Fail-open on missing IP | Deny issuance and browser jobs when IP header absent |

### VERDICT INPUT
- **Q1:** Survivable **only** with scarce issuance (1/IP/day + global cap + flag), a shrunk browser envelope (10 jobs, 1 viewport, no 4k/full_page), IP limits actually applied to capture/render (including MCP), and a flag that freezes unclaimed trials without touching claimed accounts — not as a naked auto-provision on first tool use.
- **Q2:** Claim URL/code must be a non-bearer index; finish = bound device + magic link + Authorize-with-recent-activity + browser-only nonce; refuse email collision; use the error copy above.
- **Q3:** Complement `whipnode init`; reuse magic-link + Authorize, not CLI-secret device-code; in-place same key and `mcp_devices`; recipe one-liner drops `X-API-Key` and points claim walls at `/claim`.
