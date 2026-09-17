You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The current capture handler does not establish a bounded total cost or a sub-30-second completion guarantee.
POINTER: W/routes/capture.ts:25
OPENED TEXT:
const URL_RE = /^https?:\/\/.+/;
const MAX_WAIT_MS = 25_000;

/** Hosts that are heavy SPAs — use relaxed wait strategy */
const HEAVY_SPA_HOSTS = new Set([

[2] CLAIM: The current capture handler does not establish a bounded total cost or a sub-30-second completion guarantee.
POINTER: capture.ts:695-706
OPENED TEXT:
// Enqueue for OCR + output generation (only image files need processing)
  const queueMsg: ItemProcessingMessage = {
    item_id: itemId,
    files: fileRecords
      .filter((f) => f.filename.endsWith(".png"))
      .map((f) => ({
        filename: f.filename,
        mime_type: "image/png",
        r2_key: f.r2_key,
      })),
  };
  await env.QUEUE.send(queueMsg);

  const base = env.APP_BASE_URL;
  return Response.json({

[3] CLAIM: Existing quotas cannot safely serve as trial capture admission controls.
POINTER: W/middleware/rate-limit.ts:74-78
OPENED TEXT:
const countResult = await env.DB.prepare(
    "SELECT COUNT(*) as cnt FROM items WHERE account_id = ? AND created_at > ?"
  )
    .bind(accountId, startOfDay.toISOString())
    .first<{ cnt: number }>();

  const todayUploads = countResult?.cnt ?? 0;

[4] CLAIM: Existing quotas cannot safely serve as trial capture admission controls.
POINTER: W/routes/capture.ts:234
OPENED TEXT:
// Daily upload limit
  const dailyLimit = await checkDailyUploadLimit(auth.accountId, auth.plan, env);
  if (!dailyLimit.allowed) {
    return apiError(ErrorCodes.RATE_LIMITED, dailyLimit.reason!);
  }

[5] CLAIM: Existing quotas cannot safely serve as trial capture admission controls.
POINTER: W/routes/mcp.ts:319-320
OPENED TEXT:
}
    const results = await Promise.all(
      body.map((req: unknown) => handleSingleRequest(req, env, auth))
    );
    const responses = results.filter((r): r is JsonRpcResponse => r !== null);
    response = responses.length === 0

[6] CLAIM: No stable position on “abuse cost exceeds abuse value”; a capped experiment is defensible, unrestricted renewable trial credentials are not.
POINTER: P/constants.ts:105-117
OPENED TEXT:
export const ABUSE_BUDGET = {
  /** Max free upload payload size in bytes (25 MiB) */
  MAX_FREE_UPLOAD_BYTES: 25 * 1024 * 1024,
  /** Max downloads per item before rate-limiting kicks in */
  MAX_DOWNLOADS_PER_ITEM_PER_HOUR: 200,
  /** Max OCR runs per anonymous IP per day */
  MAX_OCR_RUNS_PER_IP_PER_DAY: 100,
  /** Max bandwidth per anonymous IP per hour (500 MiB) */
  MAX_BANDWIDTH_PER_IP_PER_HOUR: 500 * 1024 * 1024,
  /** Max uploads per anonymous IP per hour */
  MAX_UPLOADS_PER_IP_PER_HOUR: 20,
  /** Max items in processing state per IP (prevents queue flooding) */
  MAX_CONCURRENT_PROCESSING_PER_IP: 5,
} as const;

// ── Content policy ────────────────────────────────────────────

[7] CLAIM: No stable position on “abuse cost exceeds abuse value”; a capped experiment is defensible, unrestricted renewable trial credentials are not.
POINTER: W/middleware/rate-limit.ts:16-58
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
    "SELECT COUNT(*) as cnt FROM items WHERE source_ip_hash = ? AND status = 'processing'"
  )
    .bind(ipHash)
    .first<{ cnt: number }>();

  const processing = processingResult?.cnt ?? 0;

  if (processing >= ABUSE_BUDGET.MAX_CONCURRENT_PROCESSING_PER_IP) {
    return {
      allowed: false,
      rea

[8] CLAIM: “No email required” is supportable as a redesign, but “zero PII stored” is not established.
POINTER: apps/worker/migrations/0001_initial_schema.sql:9
OPENED TEXT:
id            TEXT PRIMARY KEY,
  email         TEXT NOT NULL UNIQUE,
  created_at    TEXT NOT NULL,
  stripe_customer_id TEXT,
  plan          TEXT NOT NULL DEFAULT 'free',

[9] CLAIM: “No email required” is supportable as a redesign, but “zero PII stored” is not established.
POINTER: W/routes/capture.ts:606
OPENED TEXT:
const screenshotBytes = (await page.screenshot({ type: "png", fullPage })) as Uint8Array;
      await page.close();

      const filename = `capture-${vpName}.png`;

[10] CLAIM: “No email required” is supportable as a redesign, but “zero PII stored” is not established.
POINTER: packages/shared/src/hash.ts:20-25
OPENED TEXT:
export async function hashIp(ip: string): Promise<string> {
  return sha256Hex(`ip:${ip}`);
}

export async function hashUserAgent(ua: string): Promise<string> {
  return sha256Hex(`ua:${ua}`);
}

/**

[11] CLAIM: A transcript-visible link plus approval by the originating device does not defeat attacker-originated claiming.
POINTER: W/routes/auth-verify.ts:38-50
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

[12] CLAIM: Claiming free cannot honestly promise recovery of expired content or longer retention.
POINTER: P/constants.ts:43-51
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

[13] CLAIM: Claiming free cannot honestly promise recovery of expired content or longer retention.
POINTER: W/scheduled/cleanup.ts:7-8
OPENED TEXT:
/**
 * Scheduled cleanup: runs every 15 minutes.
 * Finds expired items, deletes R2 objects, marks items as deleted.
 */
export async function handleScheduled(env: Env): Promise<void> {
  const now = new Date().toISOString();

[14] CLAIM: Rung 0 should complement `init`, reusing enrollment and authentication components while adding a separate claim transaction.
POINTER: W/routes/mcp.ts:333-386
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

[15] CLAIM: Rung 0 should complement `init`, reusing enrollment and authentication components while adding a separate claim transaction.
POINTER: W/routes/auth-verify.ts:19-23
OPENED TEXT:
// Atomically mark token as used — the WHERE clause prevents TOCTOU races
  // where two concurrent requests could both read used_at IS NULL before either writes.
  const result = await env.DB.prepare(
    "UPDATE magic_tokens SET used_at = ? WHERE token_hash = ? AND used_at IS NULL AND expires_at > ?"
  ).bind(now, tokenHash, now).run();

  if (!result.meta.changes || result.meta.changes === 0) {
    return apiError(ErrorCodes.UNAUTHORIZED, "Invalid, expired, or already used link.");

[16] CLAIM: Rung 0 should complement `init`, reusing enrollment and authentication components while adding a separate claim transaction.
POINTER: apps/web/app/routes/index.tsx:181-184
OPENED TEXT:
<h2>One command to connect</h2>
        <pre><code>{`claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header "X-API-Key: YOUR_KEY" -s user`}</code></pre>
        <p className="install-note">
          <a href="/login">Create an account</a> to get your API key.
          50 uploads/day, OCR, AI analysis, and full MCP integration - no credit card needed.
        </p>
      </section>

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }