You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: A single `whipnode_capture` can pin Cloudflare Browser Rendering for up to 25s per viewport (plus a 3s heavy-SPA settle) across as many as 6 viewports including 4k and `full_page`, and that cost is **not** bounded by `ABUSE_BUDGET` IP caps because `handleCapture` never calls `checkUploadRateLimit`.
POINTER: capture.ts:25
OPENED TEXT:
const URL_RE = /^https?:\/\/.+/;
const MAX_WAIT_MS = 25_000;

/** Hosts that are heavy SPAs — use relaxed wait strategy */
const HEAVY_SPA_HOSTS = new Set([

[2] CLAIM: A single `whipnode_capture` can pin Cloudflare Browser Rendering for up to 25s per viewport (plus a 3s heavy-SPA settle) across as many as 6 viewports including 4k and `full_page`, and that cost is **not** bounded by `ABUSE_BUDGET` IP caps because `handleCapture` never calls `checkUploadRateLimit`.
POINTER: capture.ts:14-21
OPENED TEXT:
const VIEWPORTS: Record<string, { width: number; height: number; deviceScaleFactor?: number; isMobile?: boolean }> = {
  desktop:    { width: 1280, height: 800 },
  laptop:     { width: 1024, height: 768 },
  tablet:     { width: 768, height: 1024, isMobile: true },
  mobile:     { width: 375, height: 812, isMobile: true, deviceScaleFactor: 2 },
  "1080p":    { width: 1920, height: 1080 },
  "4k":       { width: 3840, height: 2160 },
};

const VALID_VIEWPORTS = new Set(Object.keys(VIEWPORTS));
const URL_RE = /^https?:\/\/.+/;

[3] CLAIM: A single `whipnode_capture` can pin Cloudflare Browser Rendering for up to 25s per viewport (plus a 3s heavy-SPA settle) across as many as 6 viewports including 4k and `full_page`, and that cost is **not** bounded by `ABUSE_BUDGET` IP caps because `handleCapture` never calls `checkUploadRateLimit`.
POINTER: capture.ts:546-548
OPENED TEXT:
}
  if (viewportNames.length > 6) {
    return apiError(ErrorCodes.VALIDATION_ERROR, "Maximum 6 viewports per capture.");
  }

  const fullPage = body.full_page ?? false;
  const plan = auth.plan;

[4] CLAIM: A single `whipnode_capture` can pin Cloudflare Browser Rendering for up to 25s per viewport (plus a 3s heavy-SPA settle) across as many as 6 viewports including 4k and `full_page`, and that cost is **not** bounded by `ABUSE_BUDGET` IP caps because `handleCapture` never calls `checkUploadRateLimit`.
POINTER: capture.ts:576-578
OPENED TEXT:
if (isHeavySpa) {
        // Heavy SPAs keep connections alive — use networkidle2 + extra settle time
        await page.goto(url, { waitUntil: "networkidle2", timeout: MAX_WAIT_MS });
        // Give the SPA extra time to render after network settles
        await new Promise((r) => setTimeout(r, 3000));
      } else {
        // Standard sites — try networkidle0, fall back to networkidle2

[5] CLAIM: A single `whipnode_capture` can pin Cloudflare Browser Rendering for up to 25s per viewport (plus a 3s heavy-SPA settle) across as many as 6 viewports including 4k and `full_page`, and that cost is **not** bounded by `ABUSE_BUDGET` IP caps because `handleCapture` never calls `checkUploadRateLimit`.
POINTER: capture.ts:233-237
OPENED TEXT:
// Daily upload limit
  const dailyLimit = await checkDailyUploadLimit(auth.accountId, auth.plan, env);
  if (!dailyLimit.allowed) {
    return apiError(ErrorCodes.RATE_LIMITED, dailyLimit.reason!);
  }

  let body: { url?: string; note?: string; viewport?: string; viewports?: string[]; full_page?: boolean; ttl_hours?: number };
  try {

[6] CLAIM: A single `whipnode_capture` can pin Cloudflare Browser Rendering for up to 25s per viewport (plus a 3s heavy-SPA settle) across as many as 6 viewports including 4k and `full_page`, and that cost is **not** bounded by `ABUSE_BUDGET` IP caps because `handleCapture` never calls `checkUploadRateLimit`.
POINTER: rate-limit.ts:16-40
OPENED TEXT:
*/
export async function checkUploadRateLimit(
  request: Request,
  env: Env
): Promise<RateLimitResult> {
  const ip = request.headers.get("CF-Connecting-IP");
  if (!ip) return { allowed: true };

  const ipHash = await hashIp(ip);

  // Check uploads in the last hour
  const oneHourAgo = new Date(Date.now() - 3600_000).toISOString();
  const countResult = await env.DB.prepare(
    "SELECT COUNT(*) as cnt FROM items WHERE source_ip_hash = ? AND created_at > ?"
  )
    .bind(ipHash, oneHourAgo)
    .first<{ cnt: number }>();

  const recentUploads = countResult?.cnt ?? 0;

  if (recentUploads >= ABUSE_BUDGET.MAX_UPLOADS_PER_IP_PER_HOUR) {
    return {
      allowed: false,
      reason: `Rate limit exceeded. Maximum ${ABUSE_BUDGET.MAX_UPLOADS_PER_IP_PER_HOUR} uploads per hour.`,
    };
  }

  // Check concurrent processing items
  const processingResult = await env.DB.prepare(

[7] CLAIM: A single `whipnode_capture` can pin Cloudflare Browser Rendering for up to 25s per viewport (plus a 3s heavy-SPA settle) across as many as 6 viewports including 4k and `full_page`, and that cost is **not** bounded by `ABUSE_BUDGET` IP caps because `handleCapture` never calls `checkUploadRateLimit`.
POINTER: items-create.ts:70
OPENED TEXT:
// Rate limit check
  const rateLimit = await checkUploadRateLimit(request, env);
  if (!rateLimit.allowed) {
    return apiError(ErrorCodes.RATE_LIMITED, rateLimit.reason!);
  }

[8] CLAIM: A single `whipnode_capture` can pin Cloudflare Browser Rendering for up to 25s per viewport (plus a 3s heavy-SPA settle) across as many as 6 viewports including 4k and `full_page`, and that cost is **not** bounded by `ABUSE_BUDGET` IP caps because `handleCapture` never calls `checkUploadRateLimit`.
POINTER: mcp.ts:1009-1013
OPENED TEXT:
// Build an internal Request and call handleCapture directly
  const fakeRequest = new Request("https://internal/api/v1/captures", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body),
  });

  // Use a no-op execution context for the direct call
  const ctx = { waitUntil: () => {}, passThroughOnException: () => {} } as unknown as ExecutionContext;

[9] CLAIM: Naked auto-provision on first unauthenticated MCP call is not survivable on the current schema or gates: MCP rejects missing auth, `accounts.email` is `NOT NULL UNIQUE`, `AccountPlan` has no trial, and nothing in the worker reads `request.cf` bot score or ASN.
POINTER: mcp.ts:291-293
OPENED TEXT:
if (!auth) {
    return jsonRpcError(null, -32000, "Authentication required. Pass your API key in the X-API-Key header.");
  }

  // Fix 4: per-account rate limiting
  if (!checkMcpRateLimit(auth.accountId)) {

[10] CLAIM: Naked auto-provision on first unauthenticated MCP call is not survivable on the current schema or gates: MCP rejects missing auth, `accounts.email` is `NOT NULL UNIQUE`, `AccountPlan` has no trial, and nothing in the worker reads `request.cf` bot score or ASN.
POINTER: 0001_initial_schema.sql:9
OPENED TEXT:
id            TEXT PRIMARY KEY,
  email         TEXT NOT NULL UNIQUE,
  created_at    TEXT NOT NULL,
  stripe_customer_id TEXT,
  plan          TEXT NOT NULL DEFAULT 'free',

[11] CLAIM: Rung-0 issuance must create `accounts` (nullable email, `plan='trial'`) + `api_keys` + `mcp_devices` in one D1 batch on `initialize`, return **only** `Mcp-Session-Id` (existing enrollment header), and refuse unless `FEATURE_ENABLE_TRIAL_ISSUANCE` is on, this IP hash has minted fewer than 1 trial in 24h, and a global daily trial-capture counter is under budget.
POINTER: mcp.ts:335-387
OPENED TEXT:
// and this is an initialize request, create a device and assign Mcp-Session-Id
  if (auth.type === "api_key" && isInitializeRequest(body)) {
    try {
      // Enforce device quota per API key (max 10) — atomic via conditional INSERT
      const sessionToken = generateMcpSessionToken();
      const tokenHash = await hashApiKey(sessionToken);
      const deviceName = parseDeviceName(request.headers.get("User-Agent"));
      const ip = request.headers.get("CF-Connecting-IP") ?? "";
      const ua = request.headers.get("User-Agent") ?? "";
      const now = isoNow();
      const expiresAt = new Date(Date.now() + 90 * 24 * 3600_000).toISOString(); // 90 days
      const deviceId = generateId();

      // Atomic quota enforcement: INSERT only if count < 10
      const insertResult = await env.DB.prepare(
        `INSERT INTO mcp_devices (id, account_id, api_key_id, session_token_hash,
         device_name, ip_hash, user_agent_hash, created_at, last_seen_at, expires_at, status)
         SELECT ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, 'active'
         WHERE (SELECT COUNT(*) FROM mcp_devices WHERE api_key_id = ? AND status = 'active') < 10`
      )
        .bind(
          deviceId,
          auth.

[12] CLAIM: Rung-0 issuance must create `accounts` (nullable email, `plan='trial'`) + `api_keys` + `mcp_devices` in one D1 batch on `initialize`, return **only** `Mcp-Session-Id` (existing enrollment header), and refuse unless `FEATURE_ENABLE_TRIAL_ISSUANCE` is on, this IP hash has minted fewer than 1 trial in 24h, and a global daily trial-capture counter is under budget.
POINTER: env.ts:39-41
OPENED TEXT:
// Feature flags
  FEATURE_ENABLE_ZIP_BUNDLES?: string;
  FEATURE_ENABLE_EMAIL_SHARING?: string;
  FEATURE_ENABLE_FACE_BLOCKING?: string;
}

[13] CLAIM: Rung-0 issuance must create `accounts` (nullable email, `plan='trial'`) + `api_keys` + `mcp_devices` in one D1 batch on `initialize`, return **only** `Mcp-Session-Id` (existing enrollment header), and refuse unless `FEATURE_ENABLE_TRIAL_ISSUANCE` is on, this IP hash has minted fewer than 1 trial in 24h, and a global daily trial-capture counter is under budget.
POINTER: rate-limit.ts:20-23
OPENED TEXT:
): Promise<RateLimitResult> {
  const ip = request.headers.get("CF-Connecting-IP");
  if (!ip) return { allowed: true };

  const ipHash = await hashIp(ip);

  // Check uploads in the last hour
  const oneHourAgo = new Date(Date.now() - 3600_000).toISOString();

[14] CLAIM: Rung-0 issuance must create `accounts` (nullable email, `plan='trial'`) + `api_keys` + `mcp_devices` in one D1 batch on `initialize`, return **only** `Mcp-Session-Id` (existing enrollment header), and refuse unless `FEATURE_ENABLE_TRIAL_ISSUANCE` is on, this IP hash has minted fewer than 1 trial in 24h, and a global daily trial-capture counter is under budget.
POINTER: rate-limit.ts:21
OPENED TEXT:
const ip = request.headers.get("CF-Connecting-IP");
  if (!ip) return { allowed: true };

  const ipHash = await hashIp(ip);

[15] CLAIM: The survivable trial envelope is 10 **browser jobs** (capture+render) per UTC day, forced single `desktop` viewport, `full_page` and `4k` rejected, TTL 24h (`DEFAULT_TTL_HOURS`), with file `whipnode_upload` sharing that 10; unconstrained 6×4k captures are not insurable for a small Worker+bindings bill.
POINTER: constants.ts:38
OPENED TEXT:
export const MAX_NOTE_CHARS = 4000;
export const DEFAULT_TTL_HOURS = 24;

// ── Plan limits ────────────────────────────────────────────────

[16] CLAIM: The survivable trial envelope is 10 **browser jobs** (capture+render) per UTC day, forced single `desktop` viewport, `full_page` and `4k` rejected, TTL 24h (`DEFAULT_TTL_HOURS`), with file `whipnode_upload` sharing that 10; unconstrained 6×4k captures are not insurable for a small Worker+bindings bill.
POINTER: constants.ts:43-51
OPENED TEXT:
export const PLAN_LIMITS: Record<string, PlanLimits> = {
  free: {
    uploads_per_day: 50,
    max_total_bytes_per_item: 25 * 1024 * 1024, // 25 MiB
    max_file_bytes: 25 * 1024 * 1024, // 25 MiB
    ttl_hours_max: 24,
    api_access: true,
    max_api_keys: 1,
    ocr_included: true,
    history: false,
  },
  pro: {
    uploads_per_day: 1000,

[17] CLAIM: The survivable trial envelope is 10 **browser jobs** (capture+render) per UTC day, forced single `desktop` viewport, `full_page` and `4k` rejected, TTL 24h (`DEFAULT_TTL_HOURS`), with file `whipnode_upload` sharing that 10; unconstrained 6×4k captures are not insurable for a small Worker+bindings bill.
POINTER: render.ts:18
OPENED TEXT:
const VALID_VIEWPORTS = new Set(Object.keys(VIEWPORTS));
const MAX_WAIT_MS = 15_000;

/**
 * POST /api/v1/render

[18] CLAIM: The survivable trial envelope is 10 **browser jobs** (capture+render) per UTC day, forced single `desktop` viewport, `full_page` and `4k` rejected, TTL 24h (`DEFAULT_TTL_HOURS`), with file `whipnode_upload` sharing that 10; unconstrained 6×4k captures are not insurable for a small Worker+bindings bill.
POINTER: render.ts:56
OPENED TEXT:
const viewportNames = body.viewports ?? ["desktop", "tablet", "mobile"];
  for (const v of viewportNames) {
    if (!VALID_VIEWPORTS.has(v)) {
      return apiError(ErrorCodes.VALIDATION_ERROR, `Invalid viewport: ${v}. Must be one of: ${[...VALID_VIEWPORTS].join(", ")}`);

[19] CLAIM: A claim URL that attaches email on GET (the current magic-link verify shape) is a transcript bearer and is unacceptable; the surfaced value may only be a public index `https://whipnode.com/claim` plus a short `WHIP-XXXX` user_code (15 min TTL, same as `magic_tokens`).
POINTER: auth-verify.ts:10-26
OPENED TEXT:
const url = new URL(request.url);
  const tokenRaw = url.searchParams.get("token");

  if (!tokenRaw) {
    return apiError(ErrorCodes.VALIDATION_ERROR, "Missing token.");
  }

  const tokenHash = await hashApiKey(tokenRaw);
  const now = isoNow();

  // Atomically mark token as used — the WHERE clause prevents TOCTOU races
  // where two concurrent requests could both read used_at IS NULL before either writes.
  const result = await env.DB.prepare(
    "UPDATE magic_tokens SET used_at = ? WHERE token_hash = ? AND used_at IS NULL AND expires_at > ?"
  ).bind(now, tokenHash, now).run();

  if (!result.meta.changes || result.meta.changes === 0) {
    return apiError(ErrorCodes.UNAUTHORIZED, "Invalid, expired, or already used link.");
  }

  // Fetch email from the now-consumed token

[20] CLAIM: A claim URL that attaches email on GET (the current magic-link verify shape) is a transcript bearer and is unacceptable; the surfaced value may only be a public index `https://whipnode.com/claim` plus a short `WHIP-XXXX` user_code (15 min TTL, same as `magic_tokens`).
POINTER: auth-magic-link.ts:39
OPENED TEXT:
const now = isoNow();
  const expiresAt = new Date(Date.now() + 15 * 60_000).toISOString();
  const ip = request.headers.get("CF-Connecting-IP") ?? "";
  const ipHash = ip ? await hashIp(ip) : null;

[21] CLAIM: A claim URL that attaches email on GET (the current magic-link verify shape) is a transcript bearer and is unacceptable; the surfaced value may only be a public index `https://whipnode.com/claim` plus a short `WHIP-XXXX` user_code (15 min TTL, same as `magic_tokens`).
POINTER: auth-magic-link.ts:51
OPENED TEXT:
// Send email in background — respond immediately so UI doesn't freeze
  const verifyUrl = `${env.APP_BASE_URL}/api/v1/auth/verify?token=${encodeURIComponent(tokenRaw)}`;

  if (env.RESEND_API_KEY) {
    ctx.waitUntil(

[22] CLAIM: Hijack (a) (leaked index claims the victim’s trial) and (b) (victim’s email bound to an attacker-held key) are both defeated only by a four-factor finish: bound `mcp_devices.id`, magic-link proof of email, Authorize page showing device name + last item notes/URLs, and a **browser-only** nonce that the bound device must POST; if `accounts.email` already exists, refuse in-place attach (do not merge).
POINTER: mcp.ts:348-367
OPENED TEXT:
// Atomic quota enforcement: INSERT only if count < 10
      const insertResult = await env.DB.prepare(
        `INSERT INTO mcp_devices (id, account_id, api_key_id, session_token_hash,
         device_name, ip_hash, user_agent_hash, created_at, last_seen_at, expires_at, status)
         SELECT ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, 'active'
         WHERE (SELECT COUNT(*) FROM mcp_devices WHERE api_key_id = ? AND status = 'active') < 10`
      )
        .bind(
          deviceId,
          auth.accountId,
          auth.keyId,
          tokenHash,
          deviceName,
          ip ? await hashIp(ip) : null,
          ua ? await hashUserAgent(ua) : null,
          now,
          now,
          expiresAt,
          auth.keyId
        )
        .run();

      if (!insertResult.meta.changes) {
        log(env, {

[23] CLAIM: Hijack (a) (leaked index claims the victim’s trial) and (b) (victim’s email bound to an attacker-held key) are both defeated only by a four-factor finish: bound `mcp_devices.id`, magic-link proof of email, Authorize page showing device name + last item notes/URLs, and a **browser-only** nonce that the bound device must POST; if `accounts.email` already exists, refuse in-place attach (do not merge).
POINTER: auth-magic-link.ts:23-32
OPENED TEXT:
// Rate limit: max 3 magic links per email per 15 min
  const fifteenMinAgo = new Date(Date.now() - 15 * 60_000).toISOString();
  const recentTokens = await env.DB.prepare(
    "SELECT COUNT(*) as cnt FROM magic_tokens WHERE email = ? AND created_at > ?"
  )
    .bind(email, fifteenMinAgo)
    .first<{ cnt: number }>();

  if ((recentTokens?.cnt ?? 0) >= 3) {
    return apiError(ErrorCodes.RATE_LIMITED, "Too many sign-in attempts. Try again in 15 minutes.");
  }

  // Generate token

[24] CLAIM: Hijack (a) (leaked index claims the victim’s trial) and (b) (victim’s email bound to an attacker-held key) are both defeated only by a four-factor finish: bound `mcp_devices.id`, magic-link proof of email, Authorize page showing device name + last item notes/URLs, and a **browser-only** nonce that the bound device must POST; if `accounts.email` already exists, refuse in-place attach (do not merge).
POINTER: auth-verify.ts:38-50
OPENED TEXT:
// Upsert account
  let account = await env.DB.prepare("SELECT id, plan FROM accounts WHERE email = ?")
    .bind(token.email)
    .first<{ id: string; plan: string }>();

  if (!account) {
    const accountId = generateId();
    await env.DB.prepare(
      "INSERT INTO accounts (id, email, created_at, plan, status) VALUES (?, ?, ?, 'free', 'active')"
    )
      .bind(accountId, token.email, now)
      .run();
    account = { id: accountId, plan: "free" };
  }

  // Create session

[25] CLAIM: Hijack (a) (leaked index claims the victim’s trial) and (b) (victim’s email bound to an attacker-held key) are both defeated only by a four-factor finish: bound `mcp_devices.id`, magic-link proof of email, Authorize page showing device name + last item notes/URLs, and a **browser-only** nonce that the bound device must POST; if `accounts.email` already exists, refuse in-place attach (do not merge).
POINTER: mcp.ts:282-286
OPENED TEXT:
const reason = auth.reason;
    const messages: Record<string, string> = {
      device_revoked: "This device has been revoked. Re-run the install command with your API key to reconnect: claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header \"X-API-Key: YOUR_KEY\"",
      device_expired: "This device session expired (inactive for 90+ days). Re-run the install command to reconnect: claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header \"X-API-Key: YOUR_KEY\"",
      key_revoked: "The API key used to register this device has been revoked. Create a new key at whipnode.com/dashboard#api-keys and re-run the install command.",
      account_inactive: "Your WhipNode account has been deactivated. Contact <email>.",
    };
    return jsonRpcError(null, -32000, messages[reason] ?? "Authentication failed.");
  }

[26] CLAIM: Rung 0 **complements** `whipnode init` (USER-FACT); it does not replace the consent path to a named free account. Claim reuses magic-link + the Authorize page, not CLI-secret device-code (that code would leak in the same transcript). Conversion is in-place: `UPDATE accounts SET email=?, plan='free'` keeping the same `api_keys` row and `mcp_devices` rows; do not rotate the key.
POINTER: index.tsx:181
OPENED TEXT:
<h2>One command to connect</h2>
        <pre><code>{`claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header "X-API-Key: YOUR_KEY" -s user`}</code></pre>
        <p className="install-note">
          <a href="/login">Create an account</a> to get your API key.
          50 uploads/day, OCR, AI analysis, and full MCP integration - no credit card needed.

[27] CLAIM: Rung 0 **complements** `whipnode init` (USER-FACT); it does not replace the consent path to a named free account. Claim reuses magic-link + the Authorize page, not CLI-secret device-code (that code would leak in the same transcript). Conversion is in-place: `UPDATE accounts SET email=?, plan='free'` keeping the same `api_keys` row and `mcp_devices` rows; do not rotate the key.
POINTER: docs.tsx:43
OPENED TEXT:
<p>Run this in any terminal window:</p>
        <pre><code>{`claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header "X-API-Key: YOUR_API_KEY" -s user`}</code></pre>
        <p>
          The <code>-s user</code> flag installs WhipNode globally - available in every project,
          not just the current directory. Then add WhipNode instructions to your

[28] CLAIM: Rung 0 **complements** `whipnode init` (USER-FACT); it does not replace the consent path to a named free account. Claim reuses magic-link + the Authorize page, not CLI-secret device-code (that code would leak in the same transcript). Conversion is in-place: `UPDATE accounts SET email=?, plan='free'` keeping the same `api_keys` row and `mcp_devices` rows; do not rotate the key.
POINTER: llms.txt:21
OPENED TEXT:
```
claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header "X-API-Key: YOUR_KEY"
```

Available MCP tools:

[29] CLAIM: Rung 0 **complements** `whipnode init` (USER-FACT); it does not replace the consent path to a named free account. Claim reuses magic-link + the Authorize page, not CLI-secret device-code (that code would leak in the same transcript). Conversion is in-place: `UPDATE accounts SET email=?, plan='free'` keeping the same `api_keys` row and `mcp_devices` rows; do not rotate the key.
POINTER: 0006_mcp_devices.sql:3-7
OPENED TEXT:
CREATE TABLE IF NOT EXISTS mcp_devices (
  id                TEXT PRIMARY KEY,
  account_id        TEXT NOT NULL,
  api_key_id        TEXT NOT NULL,
  session_token_hash TEXT NOT NULL UNIQUE,
  device_name       TEXT NOT NULL,
  ip_hash           TEXT,
  user_agent_hash   TEXT,
  created_at        TEXT NOT NULL,

[30] CLAIM: Rung 0 **complements** `whipnode init` (USER-FACT); it does not replace the consent path to a named free account. Claim reuses magic-link + the Authorize page, not CLI-secret device-code (that code would leak in the same transcript). Conversion is in-place: `UPDATE accounts SET email=?, plan='free'` keeping the same `api_keys` row and `mcp_devices` rows; do not rotate the key.
POINTER: api-keys.ts:139-143
OPENED TEXT:
.run();
    // Cascade: revoke all MCP devices enrolled by this key
    try {
      await env.DB.prepare("UPDATE mcp_devices SET status = 'revoked' WHERE api_key_id = ? AND status = 'active'")
        .bind(keyId)
        .run();
    } catch (err) {
      console.error(`Failed to cascade-revoke devices for key ${keyId}:`, err);
    }

[31] CLAIM: The kill switch is `FEATURE_ENABLE_TRIAL_ISSUANCE` off: unauthenticated `initialize` returns today’s auth error pointing at `whipnode init` / login; existing `plan='trial' AND email IS NULL` rows may read/fetch but `handleCapture`/`handleRender` reject; rows with email set and `plan` in `{free,pro,team,owner}` are untouched.
POINTER: env.ts:39-41
OPENED TEXT:
// Feature flags
  FEATURE_ENABLE_ZIP_BUNDLES?: string;
  FEATURE_ENABLE_EMAIL_SHARING?: string;
  FEATURE_ENABLE_FACE_BLOCKING?: string;
}

[32] CLAIM: The kill switch is `FEATURE_ENABLE_TRIAL_ISSUANCE` off: unauthenticated `initialize` returns today’s auth error pointing at `whipnode init` / login; existing `plan='trial' AND email IS NULL` rows may read/fetch but `handleCapture`/`handleRender` reject; rows with email set and `plan` in `{free,pro,team,owner}` are untouched.
POINTER: capture.ts:229-231
OPENED TEXT:
// Captures require authentication
  if (!auth) {
    return apiError(ErrorCodes.UNAUTHORIZED, "Authentication required. URL captures require an API key.");
  }

  // Daily upload limit
  const dailyLimit = await checkDailyUploadLimit(auth.accountId, auth.plan, env);

[33] CLAIM: The kill switch is `FEATURE_ENABLE_TRIAL_ISSUANCE` off: unauthenticated `initialize` returns today’s auth error pointing at `whipnode init` / login; existing `plan='trial' AND email IS NULL` rows may read/fetch but `handleCapture`/`handleRender` reject; rows with email set and `plan` in `{free,pro,team,owner}` are untouched.
POINTER: render.ts:33-35
OPENED TEXT:
): Promise<Response> {
  if (!auth) {
    return apiError(ErrorCodes.UNAUTHORIZED, "Authentication required.");
  }

  let body: {
    item_id?: string;

[34] CLAIM: The kill switch is `FEATURE_ENABLE_TRIAL_ISSUANCE` off: unauthenticated `initialize` returns today’s auth error pointing at `whipnode init` / login; existing `plan='trial' AND email IS NULL` rows may read/fetch but `handleCapture`/`handleRender` reject; rows with email set and `plan` in `{free,pro,team,owner}` are untouched.
POINTER: mcp.ts:292
OPENED TEXT:
if (!auth) {
    return jsonRpcError(null, -32000, "Authentication required. Pass your API key in the X-API-Key header.");
  }

  // Fix 4: per-account rate limiting

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }