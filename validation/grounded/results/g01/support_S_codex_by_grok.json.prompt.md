You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: Reuse magic-link email proof, but defer new-account creation until an explicit browser approval.
POINTER: W/routes/auth-verify.ts:43-50
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

[2] CLAIM: Existing MCP enrollment provides reusable device records, but does not provide unauthenticated device-code onboarding.
POINTER: W/routes/mcp.ts:291-292
OPENED TEXT:
if (!auth) {
    return jsonRpcError(null, -32000, "Authentication required. Pass your API key in the X-API-Key header.");
  }

  // Fix 4: per-account rate limiting

[3] CLAIM: Device revocation must invalidate its dedicated API key as well to prevent reconnection using retained credentials.
POINTER: W/routes/mcp-devices.ts:70
OPENED TEXT:
await env.DB.prepare("UPDATE mcp_devices SET status = 'revoked' WHERE id = ?")
    .bind(deviceId)
    .run();

[4] CLAIM: Device revocation must invalidate its dedicated API key as well to prevent reconnection using retained credentials.
POINTER: W/routes/mcp.ts:335
OPENED TEXT:
// and this is an initialize request, create a device and assign Mcp-Session-Id
  if (auth.type === "api_key" && isInitializeRequest(body)) {
    try {
      // Enforce device quota per API key (max 10) — atomic via conditional INSERT
      const sessionToken = generateMcpSessionToken();

[5] CLAIM: Per-device credentials should segregate installation access without promising private per-device content.
POINTER: W/routes/mcp.ts:1151
OPENED TEXT:
conditions.push(
        "(device_id IN (SELECT id FROM mcp_devices WHERE account_id = ? AND group_name = ?) OR device_id IS NULL)"
      );
      binds.push(auth.accountId, thisDevice.group_name);
    }

[6] CLAIM: Per-device credentials should segregate installation access without promising private per-device content.
POINTER: W/routes/items-get.ts:7-13
OPENED TEXT:
export async function handleGetItem(
  itemId: string,
  env: Env
): Promise<Response> {
  const row = await env.DB.prepare("SELECT * FROM items WHERE id = ?")
    .bind(itemId)
    .first<ItemRow>();

  if (!row) {
    return apiError(ErrorCodes.ITEM_NOT_FOUND, "Item not found.");

[7] CLAIM: Capture admission needs durable reservations before browser work rather than relying on completed-item counts.
POINTER: W/middleware/rate-limit.ts:74-78
OPENED TEXT:
const countResult = await env.DB.prepare(
    "SELECT COUNT(*) as cnt FROM items WHERE account_id = ? AND created_at > ?"
  )
    .bind(accountId, startOfDay.toISOString())
    .first<{ cnt: number }>();

  const todayUploads = countResult?.cnt ?? 0;

[8] CLAIM: Capture admission needs durable reservations before browser work rather than relying on completed-item counts.
POINTER: W/routes/capture.ts:565
OPENED TEXT:
try {
    const browser = await puppeteer.launch(env.BROWSER);

    for (let i = 0; i < viewportNames.length; i++) {
      const vpName = viewportNames[i];

[9] CLAIM: Email verification alone is an insufficient proposed farming defense.
POINTER: W/routes/auth-magic-link.ts:23-32
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

[10] CLAIM: Published entitlements must be reconciled before introducing capture-specific limits.
POINTER: apps/web/app/routes/pricing.tsx:18-19
OPENED TEXT:
<p className="pricing-subtitle">
        WhipNode is in early access. All early adopters get the free plan - no limits will change
        without notice.
      </p>

      <div className="pricing-early-banner">

[11] CLAIM: Published entitlements must be reconciled before introducing capture-specific limits.
POINTER: P/constants.ts:55
OPENED TEXT:
uploads_per_day: 1000,
    max_total_bytes_per_item: 250 * 1024 * 1024, // 250 MiB
    max_file_bytes: 50 * 1024 * 1024, // 50 MiB per file
    ttl_hours_max: 168,
    api_access: true,

[12] CLAIM: Data minimization requires cleanup changes, beyond configuring a 24-hour TTL.
POINTER: W/scheduled/cleanup.ts:22-24
OPENED TEXT:
if (!expired.results || expired.results.length === 0) {
    return;
  }

  for (const item of expired.results) {
    await cleanupItem(item.id, env);

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }