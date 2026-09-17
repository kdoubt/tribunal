You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: MCP and docs today cannot take an agent from nothing to a capture: unauthenticated MCP returns JSON-RPC `-32000` asking for `X-API-Key`, and Getting Started is login → create key → paste `claude mcp add … --header "X-API-Key: …"`.
POINTER: apps/worker/src/routes/mcp.ts:291-292
OPENED TEXT:
if (!auth) {
    return jsonRpcError(null, -32000, "Authentication required. Pass your API key in the X-API-Key header.");
  }

  // Fix 4: per-account rate limiting

[2] CLAIM: MCP and docs today cannot take an agent from nothing to a capture: unauthenticated MCP returns JSON-RPC `-32000` asking for `X-API-Key`, and Getting Started is login → create key → paste `claude mcp add … --header "X-API-Key: …"`.
POINTER: apps/web/app/routes/docs.tsx:29-43
OPENED TEXT:
<h3>Step 1 - Create an account</h3>
        <p>
          <a href="/login">Sign in</a> with your email. No password needed - we
          send you a magic link.
        </p>

        <h3>Step 2 - Create an API key</h3>
        <p>
          Go to the <a href="/dashboard#api-keys">API Keys tab</a> in your
          dashboard and create a key. Save it - it's only shown once.
        </p>

        <h3>Step 3 - Install</h3>
        <p>Run this in any terminal window:</p>
        <pre><code>{`claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header "X-API-Key: YOUR_API_KEY" -s user`}</code></pre>
        <p>
          The <code>-s user</code> flag installs WhipNode globally - available in every project,
          not just the current directory. Then add WhipNode instructions to your

[3] CLAIM: MCP and docs today cannot take an agent from nothing to a capture: unauthenticated MCP returns JSON-RPC `-32000` asking for `X-API-Key`, and Getting Started is login → create key → paste `claude mcp add … --header "X-API-Key: …"`.
POINTER: apps/worker/src/routes/capture.ts:229-231
OPENED TEXT:
// Captures require authentication
  if (!auth) {
    return apiError(ErrorCodes.UNAUTHORIZED, "Authentication required. URL captures require an API key.");
  }

  // Daily upload limit
  const dailyLimit = await checkDailyUploadLimit(auth.accountId, auth.plan, env);

[4] CLAIM: MCP and docs today cannot take an agent from nothing to a capture: unauthenticated MCP returns JSON-RPC `-32000` asking for `X-API-Key`, and Getting Started is login → create key → paste `claude mcp add … --header "X-API-Key: …"`.
POINTER: docs/ROADMAP.md:36
OPENED TEXT:
- ShareX target uploader
- CLI tool for terminal workflows
- Import adapters (Google Drive, Slack, etc.)

### Protocol Evolution

[5] CLAIM: Do not issue anonymous trial keys or auto-create accounts; the single consent moment is a human click on `/init` “Authorize this device” after a session cookie or magic-link verify, which upserts `accounts.plan='free'` and enrolls the device.
POINTER: apps/worker/src/routes/auth-verify.ts:43-50
OPENED TEXT:
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

[6] CLAIM: Do not issue anonymous trial keys or auto-create accounts; the single consent moment is a human click on `/init` “Authorize this device” after a session cookie or magic-link verify, which upserts `accounts.plan='free'` and enrolls the device.
POINTER: apps/worker/src/lib/email.ts:23-32
OPENED TEXT:
to: [email],
        subject: "Sign in to WhipNode",
        text: `Sign in to WhipNode by clicking this link:\n\n${verifyUrl}\n\nThis link expires in 15 minutes. If you didn't request this, ignore this email.`,
        html: `
          <div style="font-family: -apple-system, sans-serif; max-width: 480px; margin: 0 auto; padding: 32px 0;">
            <h2 style="font-size: 18px; color: #1a1a1a; margin-bottom: 16px;">Sign in to WhipNode</h2>
            <p style="font-size: 15px; color: #444; line-height: 1.6; margin-bottom: 24px;">
              Click the button below to sign in. This link expires in 15 minutes.
            </p>
            <a href="${verifyUrl}" style="display: inline-block; background: #2D5F73; color: #fff; padding: 12px 28px; text-decoration: none; font-size: 14px; font-weight: 600;">
              Sign in to WhipNode
            </a>
            <p style="font-size: 13px; color: #888; margin-top: 24px; line-height: 1.5;">
              If you didn't request this email, you can safely ignore it.

[7] CLAIM: Build a short-lived device-code pairing API + `/init` page (new); reuse magic-link (15 min, 3/email/15 min), one `api_keys` row, and `mcp_devices` insert/session (90-day sliding, max 10 active per key) instead of OAuth.
POINTER: apps/worker/src/routes/auth-magic-link.ts:23-32
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

[8] CLAIM: Build a short-lived device-code pairing API + `/init` page (new); reuse magic-link (15 min, 3/email/15 min), one `api_keys` row, and `mcp_devices` insert/session (90-day sliding, max 10 active per key) instead of OAuth.
POINTER: apps/worker/src/routes/mcp.ts:334-367
OPENED TEXT:
// Device enrollment: if authenticated via API key (not yet a device session)
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
         WHERE (SELECT COUNT(*) FROM mcp_devices WHERE api_key_id = ? AND sta

[9] CLAIM: Build a short-lived device-code pairing API + `/init` page (new); reuse magic-link (15 min, 3/email/15 min), one `api_keys` row, and `mcp_devices` insert/session (90-day sliding, max 10 active per key) instead of OAuth.
POINTER: apps/worker/src/middleware/auth.ts:114-115
OPENED TEXT:
// Update last_seen and extend session (sliding window — active devices never expire)
  const newExpiry = new Date(Date.now() + 90 * 24 * 3600_000).toISOString();
  env.DB.prepare("UPDATE mcp_devices SET last_seen_at = datetime('now'), expires_at = ? WHERE id = ?")
    .bind(newExpiry, row.device_id)
    .run()

[10] CLAIM: Free-tier segregation is per-device `mcp_devices` sessions, not per-device API keys: `PLAN_LIMITS.free.max_api_keys` is 1 and create-key 403s at the cap; pairing must reuse/create that one `["write"]` key and return a device session (raw keys are shown once and only hashed thereafter).
POINTER: packages/protocol/src/constants.ts:42-51
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

[11] CLAIM: Free-tier segregation is per-device `mcp_devices` sessions, not per-device API keys: `PLAN_LIMITS.free.max_api_keys` is 1 and create-key 403s at the cap; pairing must reuse/create that one `["write"]` key and return a device session (raw keys are shown once and only hashed thereafter).
POINTER: apps/worker/src/routes/api-keys.ts:42-46
OPENED TEXT:
if ((existing?.cnt ?? 0) >= maxKeys) {
    return apiError(
      ErrorCodes.FORBIDDEN,
      `Your account allows ${maxKeys} API key(s). Revoke an existing key or upgrade.`
    );
  }

  const { raw, prefix } = generateApiKey();

[12] CLAIM: Free-tier segregation is per-device `mcp_devices` sessions, not per-device API keys: `PLAN_LIMITS.free.max_api_keys` is 1 and create-key 403s at the cap; pairing must reuse/create that one `["write"]` key and return a device session (raw keys are shown once and only hashed thereafter).
POINTER: apps/web/app/components/ApiKeyManager.tsx:309-310
OPENED TEXT:
{createdKey && (
        <div className="created-key-banner">
          <p><strong>Save this key now - it won't be shown again.</strong></p>
          <div className="copyable">
            <code>{createdKey}</code>
            <button onClick={copyKey}>

[13] CLAIM: Free-tier segregation is per-device `mcp_devices` sessions, not per-device API keys: `PLAN_LIMITS.free.max_api_keys` is 1 and create-key 403s at the cap; pairing must reuse/create that one `["write"]` key and return a device session (raw keys are shown once and only hashed thereafter).
POINTER: apps/worker/src/middleware/auth.ts:26-52
OPENED TEXT:
*/
export async function resolveAuth(
  request: Request,
  env: Env
): Promise<AuthContext> {
  // 1. Check Mcp-Session-Id header (MCP device sessions)
  const mcpSessionId = request.headers.get("Mcp-Session-Id");
  if (mcpSessionId) {
    const device = await resolveMcpDevice(mcpSessionId, env);
    if (device) {
      if (device.type === "rejected") {
        // Expired devices silently fall through to API key re-enrollment
        if (device.reason !== "device_expired") {
          return device; // Return rejection for revoked/inactive
        }
        // Fall through to API key for expired devices
      } else {
        return device;
      }
    }
    // If device session unknown or expired, fall through to API key
  }

  // 2. Check X-API-Key header
  const apiKey = request.headers.get("X-API-Key");
  if (apiKey) {
    return resolveApiKey(apiKey, env);
  }

  // 3. Check session cookie
  const cookie = request.headers.get("Cookie");

[14] CLAIM: Shape free for this funnel as: 50 uploads/day, OCR, 25 MB, 24 h TTL, 1 API key, 1 viewport per capture, ~50 captures/month; Pro walls are extra viewports/4k, 50 MB/file, 7 d TTL, history, 10 keys — viewports are not plan-gated today (`max 6` only).
POINTER: packages/protocol/src/constants.ts:42-62
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
    max_total_bytes_per_item: 250 * 1024 * 1024, // 250 MiB
    max_file_bytes: 50 * 1024 * 1024, // 50 MiB per file
    ttl_hours_max: 168,
    api_access: true,
    max_api_keys: 10,
    ocr_included: true,
    history: true,
  },
  team: {
    uploads_per_day: 5000,
    max_total_bytes_per_item: 250 * 1024 * 1024, // 250 MiB

[15] CLAIM: Shape free for this funnel as: 50 uploads/day, OCR, 25 MB, 24 h TTL, 1 API key, 1 viewport per capture, ~50 captures/month; Pro walls are extra viewports/4k, 50 MB/file, 7 d TTL, history, 10 keys — viewports are not plan-gated today (`max 6` only).
POINTER: apps/web/app/routes/pricing.tsx:27-53
OPENED TEXT:
<div className="pricing-grid">
        <div className="plan-card plan-current">
          <h2>Early Access</h2>
          <p className="plan-price">$0</p>
          <ul>
            <li>50 uploads/day</li>
            <li>25 MB per item</li>
            <li>24h retention</li>
            <li>OCR text extraction</li>
            <li>5 output formats</li>
            <li>1 API key</li>
          </ul>
          <a href="/new" className="btn btn-primary">
            Get Started
          </a>
        </div>

        <div className="plan-card plan-disabled">
          <h2>Pro</h2>
          <p className="plan-price plan-price-tbd">TBD</p>
          <ul>
            <li>50 MB per file, 100 MB per item</li>
            <li>Higher upload limits</li>
            <li>Extended retention</li>
            <li>Priority processing</li>
            <li>Upload history</li>
            <li>Advanced analytics</li>
          </ul>
        </div>

        <div className="plan-card plan-disabled">

[16] CLAIM: Shape free for this funnel as: 50 uploads/day, OCR, 25 MB, 24 h TTL, 1 API key, 1 viewport per capture, ~50 captures/month; Pro walls are extra viewports/4k, 50 MB/file, 7 d TTL, history, 10 keys — viewports are not plan-gated today (`max 6` only).
POINTER: apps/worker/src/routes/capture.ts:536-548
OPENED TEXT:
// Support single viewport (backward compat) or multiple viewports
  const viewportNames = body.viewports
    ?? (body.viewport ? [body.viewport] : ["desktop"]);
  for (const v of viewportNames) {
    if (!VALID_VIEWPORTS.has(v)) {
      return apiError(
        ErrorCodes.VALIDATION_ERROR,
        `Invalid viewport: ${v}. Must be one of: ${[...VALID_VIEWPORTS].join(", ")}`
      );
    }
  }
  if (viewportNames.length > 6) {
    return apiError(ErrorCodes.VALIDATION_ERROR, "Maximum 6 viewports per capture.");
  }

  const fullPage = body.full_page ?? false;
  const plan = auth.plan;

[17] CLAIM: Shape free for this funnel as: 50 uploads/day, OCR, 25 MB, 24 h TTL, 1 API key, 1 viewport per capture, ~50 captures/month; Pro walls are extra viewports/4k, 50 MB/file, 7 d TTL, history, 10 keys — viewports are not plan-gated today (`max 6` only).
POINTER: apps/web/app/routes/terms.tsx:61
OPENED TEXT:
<li>OCR included</li>
        <li>No API access (browser and paste only)</li>
        <li>No upload history</li>
      </ul>
      <p>

[18] CLAIM: Shape free for this funnel as: 50 uploads/day, OCR, 25 MB, 24 h TTL, 1 API key, 1 viewport per capture, ~50 captures/month; Pro walls are extra viewports/4k, 50 MB/file, 7 d TTL, history, 10 keys — viewports are not plan-gated today (`max 6` only).
POINTER: login.tsx:94-95
OPENED TEXT:
<p className="auth-hint">
        Free accounts include 1 API key for agent access, 50 uploads/day, and OCR.
      </p>
    </div>
  );
}

[19] CLAIM: For users who never open the dashboard, the upsell is the existing limit error that names the plan plus `whipnode_status`’s `Account: … (plan)` line — one sentence, only on the wall; the methodology recipe lives on whipnode.com (init URL, what Authorize grants, one capture example, free limits, `/pricing`), not in the unnamed repo.
POINTER: apps/worker/src/middleware/rate-limit.ts:82-85
OPENED TEXT:
if (todayUploads >= limits.uploads_per_day) {
    return {
      allowed: false,
      reason: `Daily upload limit reached. Your ${plan} plan allows ${limits.uploads_per_day} uploads per day.`,
    };
  }

[20] CLAIM: For users who never open the dashboard, the upsell is the existing limit error that names the plan plus `whipnode_status`’s `Account: … (plan)` line — one sentence, only on the wall; the methodology recipe lives on whipnode.com (init URL, what Authorize grants, one capture example, free limits, `/pricing`), not in the unnamed repo.
POINTER: apps/worker/src/routes/mcp.ts:672-673
OPENED TEXT:
const lines: string[] = [];
  lines.push(`WhipNode Status`);
  lines.push(`Account: ${account?.email ?? "unknown"} (${account?.plan ?? "free"} plan)`);
  lines.push(`Uploads today: ${uploadCount?.cnt ?? 0}`);
  lines.push("");

[21] CLAIM: For users who never open the dashboard, the upsell is the existing limit error that names the plan plus `whipnode_status`’s `Account: … (plan)` line — one sentence, only on the wall; the methodology recipe lives on whipnode.com (init URL, what Authorize grants, one capture example, free limits, `/pricing`), not in the unnamed repo.
POINTER: apps/web/app/routes/pricing.tsx:43-45
OPENED TEXT:
<div className="plan-card plan-disabled">
          <h2>Pro</h2>
          <p className="plan-price plan-price-tbd">TBD</p>
          <ul>
            <li>50 MB per file, 100 MB per item</li>
            <li>Higher upload limits</li>

[22] CLAIM: Email verify is necessary and not sufficient against farming: magic-link has no CAPTCHA, only `COUNT(*) … >= 3` per email; add Turnstile on `/init`, cap free active devices to 3 (today 10/key), 10-minute pairing TTL, keep hashed IP; the agent may start/poll pairing and capture after, and must not read mail or click the link.
POINTER: apps/worker/src/routes/auth-magic-link.ts:23-32
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

[23] CLAIM: Email verify is necessary and not sufficient against farming: magic-link has no CAPTCHA, only `COUNT(*) … >= 3` per email; add Turnstile on `/init`, cap free active devices to 3 (today 10/key), 10-minute pairing TTL, keep hashed IP; the agent may start/poll pairing and capture after, and must not read mail or click the link.
POINTER: apps/worker/src/routes/mcp.ts:347-352
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

[24] CLAIM: Email verify is necessary and not sufficient against farming: magic-link has no CAPTCHA, only `COUNT(*) … >= 3` per email; add Turnstile on `/init`, cap free active devices to 3 (today 10/key), 10-minute pairing TTL, keep hashed IP; the agent may start/poll pairing and capture after, and must not read mail or click the link.
POINTER: mcp.ts:214-217
OPENED TEXT:
},
        apply_config: {
          type: "string",
          description: "Pass the exact JSON config string returned by a previous configure proposal to apply it. Only call this AFTER showing the proposal to the user and getting their confirmation.",
        },
      },
    },
  },

[25] CLAIM: Init should store only email, API-key hash, device_name, ip_hash, user_agent_hash, and 24 h items; revocation/expiry already exist (device revoke, key cascade-revoke, 90-day inactive, sliding `expires_at`).
POINTER: docs/PRIVACY.md:3-16
OPENED TEXT:
## What is stored

- Uploaded files (in R2 object storage)
- Item metadata: ID, timestamps, status, file count, total bytes, MIME types
- Optional note text provided by the user
- OCR text extracted from images/PDFs
- Hashed IP address and user agent (for rate limiting and abuse prevention)
- For authenticated users: email, plan, API key hashes

## How long it is stored

- Files and derived outputs: until expiration (default 24 hours, max depends on plan)
- After expiration: file data is deleted from storage within 15 minutes
- Tombstone metadata (item ID, timestamps, status): may be retained for abuse prevention

## What analytics are collected

[26] CLAIM: Init should store only email, API-key hash, device_name, ip_hash, user_agent_hash, and 24 h items; revocation/expiry already exist (device revoke, key cascade-revoke, 90-day inactive, sliding `expires_at`).
POINTER: apps/worker/src/routes/mcp-devices.ts:70-74
OPENED TEXT:
await env.DB.prepare("UPDATE mcp_devices SET status = 'revoked' WHERE id = ?")
    .bind(deviceId)
    .run();

  return Response.json({ ok: true, message: "Device revoked. It will need to re-authenticate on next request." });
}

export async function handleRenameMcpDevice(

[27] CLAIM: Init should store only email, API-key hash, device_name, ip_hash, user_agent_hash, and 24 h items; revocation/expiry already exist (device revoke, key cascade-revoke, 90-day inactive, sliding `expires_at`).
POINTER: apps/worker/src/routes/api-keys.ts:139-143
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

[28] CLAIM: Init should store only email, API-key hash, device_name, ip_hash, user_agent_hash, and 24 h items; revocation/expiry already exist (device revoke, key cascade-revoke, 90-day inactive, sliding `expires_at`).
POINTER: apps/worker/src/routes/mcp.ts:282-284
OPENED TEXT:
const reason = auth.reason;
    const messages: Record<string, string> = {
      device_revoked: "This device has been revoked. Re-run the install command with your API key to reconnect: claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header \"X-API-Key: YOUR_KEY\"",
      device_expired: "This device session expired (inactive for 90+ days). Re-run the install command to reconnect: claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header \"X-API-Key: YOUR_KEY\"",
      key_revoked: "The API key used to register this device has been revoked. Create a new key at whipnode.com/dashboard#api-keys and re-run the install command.",
      account_inactive: "Your WhipNode account has been deactivated. Contact <email>.",
    };

[29] CLAIM: Init should store only email, API-key hash, device_name, ip_hash, user_agent_hash, and 24 h items; revocation/expiry already exist (device revoke, key cascade-revoke, 90-day inactive, sliding `expires_at`).
POINTER: apps/web/app/components/McpDeviceManager.tsx:77
OPENED TEXT:
<span>Registered: {new Date(d.created_at).toLocaleDateString()}</span>
                <span>Session: auto-renews on use (expires if inactive for 90 days)</span>
              </div>
              <div className="device-actions">
                {d.status === "active" && (

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }