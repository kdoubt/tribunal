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
Plausible framings: minimize setup clicks; establish a consent and credential boundary; design a monetization funnel. I choose the consent boundary first because unattended creation is prohibited (USER-FACT).

Paths below use `W = apps/worker/src/`, `P = packages/protocol/src/`, and `M = apps/worker/migrations/`, all under `artifact/`. **ASSUMPTION:** proposed behavior and numerical limits below are design recommendations requiring implementation and validation.

CLAIM: Reuse magic-link email proof, but defer new-account creation until an explicit browser approval.
EVIDENCE: `W/routes/auth-verify.ts:43–50` inserts the free account during verification; USER-FACT requires human consent.
CONFIDENCE: 0.95
FALSIFIER: An unapproved email verification cannot create an account in the deployed implementation.

CLAIM: Existing MCP enrollment provides reusable device records, but does not provide unauthenticated device-code onboarding.
EVIDENCE: `W/routes/mcp.ts:291–292` rejects missing credentials; `:335–352` enrolls through an existing API key.
CONFIDENCE: 0.97
FALSIFIER: A credential-free request completes a human-approved pairing through these handlers.

CLAIM: Device revocation must invalidate its dedicated API key as well to prevent reconnection using retained credentials.
EVIDENCE: `W/routes/mcp-devices.ts:70` updates only device status; `W/routes/mcp.ts:335` permits API-key enrollment.
CONFIDENCE: 0.95
FALSIFIER: After device revocation, the retained key alone cannot authenticate or enroll another device.

CLAIM: Per-device credentials should segregate installation access without promising private per-device content.
EVIDENCE: `W/routes/mcp.ts:1151` includes unassigned items in grouped listings; `W/routes/items-get.ts:7–13` retrieves by item ID without an identity parameter.
CONFIDENCE: 0.97
FALSIFIER: Those read paths enforce device ownership before returning content.

CLAIM: Capture admission needs durable reservations before browser work rather than relying on completed-item counts.
EVIDENCE: `W/middleware/rate-limit.ts:74–78` counts items; `W/routes/capture.ts:565` launches the browser before `:663` inserts the item.
CONFIDENCE: 0.96
FALSIFIER: Concurrent requests reserve account capacity atomically before reaching browser launch.

CLAIM: Email verification alone is an insufficient proposed farming defense.
EVIDENCE: `W/routes/auth-magic-link.ts:23–32` limits requests per email; `:40–47` records IP hashes without an IP admission check; ASSUMPTION: attackers can obtain multiple inboxes.
CONFIDENCE: 0.88
FALSIFIER: Controlled farming trials show inbox acquisition alone consistently costs more than obtainable free-service value.

CLAIM: Published entitlements must be reconciled before introducing capture-specific limits.
EVIDENCE: `apps/web/app/routes/pricing.tsx:18–19` promises notice; `:47` advertises 100 MB/item, while `P/constants.ts:55` specifies 250 MiB.
CONFIDENCE: 0.98
FALSIFIER: The published page and enforced entitlements already agree and communicate the proposed restrictions.

CLAIM: Data minimization requires cleanup changes, beyond configuring a 24-hour TTL.
EVIDENCE: `W/scheduled/cleanup.ts:22–24` returns before auth cleanup at `:47–60`; `:188–191` marks items deleted without clearing their content-bearing columns.
CONFIDENCE: 0.97
FALSIFIER: Independent cleanup reliably purges expired auth records and removes retained notes/OCR from expired items.

**FLOW SPEC — proposed / ASSUMPTION**

1. A user with nothing installs the CLI using instructions on whipnode.com and runs `whipnode init`. Dashboard copy-paste supplies this same command without credentials. Credential-free MCP first use returns a setup URL and command; it creates no account.
2. The CLI creates a pending installation and prints a matching browser code. Build a small D1 pairing table with hashed polling secret, credential hashes, expiry, and approval state; no anonymous trial key.
3. Generate the API key and device-session secret locally; transmit only hashes over HTTPS. Keep pending secrets in a user-only configuration file, outside repositories, with Unix mode `0600` or equivalent ACL.
4. Open the pairing page, with manual URL/code fallback for remote terminals. Human email entry and magic-link verification establish a short-lived verified identity, not an account. Reuse token machinery from `auth-verify.ts`; change its account-creation behavior.
5. Show email, editable installation name, matching code, free limits, expiry, local configuration changes, and public-link sharing implications. The **single consent button** says: **“Create my free account and connect this device.”** Existing users see “Connect this device.” Explain capture/render/upload/read authority and the first demo capture.
6. A CSRF-protected approval POST atomically creates or selects the account and registers one key/device pair. Name both `init/<label>/<short-id>`. Enforce account-wide caps; change the existing one-key free limit (`P/constants.ts:49`). Restrict credential-management endpoints to human sessions.
7. Poll every five seconds with backoff. Idempotent approval and polling recover connection loss without duplicate credentials. Ten-minute pending expiry, denial, and cancellation produce distinct messages; resend respects the existing three-per-15-minute email limit (`auth-magic-link.ts:23–32`).
8. On approval, activate the local profile and configure the selected MCP client. Support `WHIPNODE_API_KEY` as an explicit override; never print secrets or put them in shell history. Existing dashboard examples use that variable (`ApiKeyManager.tsx:74`).
9. Initialize using the registered device session. Enforce 90-day inactivity expiry on **both** credentials, replacing the present device-only sliding extension (`W/middleware/auth.ts:114–119`). CLI/dashboard revocation disables both; expired credentials require human reconnection.
10. Capture the disclosed demo page once. Return screenshot and expiry immediately when available; label OCR pending, as `W/routes/mcp.ts:1052–1053` already does. Retry status queries without recapturing; build capture idempotency for ambiguous timeouts.

**TIER TABLE — proposed / ASSUMPTION**

| Dimension | Free | Pro |
|---|---|---|
| Price | $0, no card | TBD; no purchase CTA before availability |
| Capture/render units | 60/month, 10/day | 1,000/month |
| Retention | 24 hours | Up to 7 days |
| Viewports | Desktop/mobile, two/request | Six/request |
| OCR | 60 image/page units/month | 1,000/month |
| Admission | 2 starts/minute; 1 concurrent | 10/minute; 3 concurrent |
| Uploads | 50/day; 25 MiB/file/item | 1,000/day; 50 MiB/file, 250 MiB/item |
| Dedicated key/device pairs | 2/account | 10/account |

Count each rendered viewport as one unit; meter OCR across uploads and captures. Add bounded pixels, browser runtime, and download budgets. Preserve existing users’ terms until notified.

**Guardrails — proposed / ASSUMPTION:** Validate Turnstile server-side before sending signup email; combine durable email/IP velocity limits, account caps, and global spend cutoffs. Use IP signals for challenges with shared-network recovery. Preserve trusted ingress metadata through MCP’s internal calls, which currently construct only a content-type header (`W/routes/mcp.ts:1009–1012`). Agents may initiate pairing, poll, configure, and perform authorized captures; humans handle email, challenges, approval, and purchases.

Store email, consent receipt, credential hashes, installation label, timestamps, usage counters, and short-lived abuse hashes. Purge pending records promptly, scrub expired content, and retain minimal abuse records for 30 days. No transcript or repository collection.

**UPSELL TRIGGERS — proposed / ASSUMPTION**

- Quota exhaustion: terminal/MCP error shows remaining units, reset time, and optional pricing link.
- Longer retention: show exact expiry and export option before upgrading.
- Additional viewports or devices: explain the specific entitlement required.
- `whipnode status`: show usage on request; no repeated promotional messages.
- Recipe page: install/demo, consent preview, limits, troubleshooting, and orchestrator-captures/static-seat-evidence example; methodology repo remains uncoupled (USER-FACT).

**VERDICT INPUT**

Q1: Prefer magic-link proof plus browser-approved device pairing; under-two-minute completion remains an unvalidated target.

Q2: Launch transparent, metered free access; offer Pro only at explicit resource limits.

Q3: Require durable admission and credential revocation fixes; no stable position on abuse-cost superiority without measured farming and workload economics.


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}