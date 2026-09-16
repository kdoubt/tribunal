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
