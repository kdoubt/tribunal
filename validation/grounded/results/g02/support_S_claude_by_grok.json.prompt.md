You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: ** One rung-0 "capture" is up to six full browser page-loads plus six OCR jobs, so a quota counted in captures understates cost ~6×.
POINTER: capture.ts:546
OPENED TEXT:
}
  if (viewportNames.length > 6) {
    return apiError(ErrorCodes.VALIDATION_ERROR, "Maximum 6 viewports per capture.");
  }

[2] CLAIM: ** One rung-0 "capture" is up to six full browser page-loads plus six OCR jobs, so a quota counted in captures understates cost ~6×.
POINTER: capture.ts:567-619
OPENED TEXT:
for (let i = 0; i < viewportNames.length; i++) {
      const vpName = viewportNames[i];
      const vp = VIEWPORTS[vpName];
      const page = await browser.newPage();
      await page.setUserAgent(CHROME_UA);
      await page.setViewport(vp);

      // Navigate with adaptive wait strategy
      if (isHeavySpa) {
        // Heavy SPAs keep connections alive — use networkidle2 + extra settle time
        await page.goto(url, { waitUntil: "networkidle2", timeout: MAX_WAIT_MS });
        // Give the SPA extra time to render after network settles
        await new Promise((r) => setTimeout(r, 3000));
      } else {
        // Standard sites — try networkidle0, fall back to networkidle2
        try {
          await page.goto(url, { waitUntil: "networkidle0", timeout: MAX_WAIT_MS });
        } catch (navErr: any) {
          if (navErr?.message?.includes("timeout")) {
            // Page loaded but still has active connections — take what we have
            console.log(`networkidle0 timeout for ${url}, proceeding with current state`);
          } else {
            throw navErr;
          }
        }
      }

      // Extract sitemap from the first viewport's page (skip for Google apps

[3] CLAIM: ** One rung-0 "capture" is up to six full browser page-loads plus six OCR jobs, so a quota counted in captures understates cost ~6×.
POINTER: capture.ts:25
OPENED TEXT:
const URL_RE = /^https?:\/\/.+/;
const MAX_WAIT_MS = 25_000;

/** Hosts that are heavy SPAs — use relaxed wait strategy */
const HEAVY_SPA_HOSTS = new Set([

[4] CLAIM: ** One rung-0 "capture" is up to six full browser page-loads plus six OCR jobs, so a quota counted in captures understates cost ~6×.
POINTER: capture.ts:579
OPENED TEXT:
// Give the SPA extra time to render after network settles
        await new Promise((r) => setTimeout(r, 3000));
      } else {
        // Standard sites — try networkidle0, fall back to networkidle2
        try {

[5] CLAIM: ** One rung-0 "capture" is up to six full browser page-loads plus six OCR jobs, so a quota counted in captures understates cost ~6×.
POINTER: capture.ts:20
OPENED TEXT:
"1080p":    { width: 1920, height: 1080 },
  "4k":       { width: 3840, height: 2160 },
};

const VALID_VIEWPORTS = new Set(Object.keys(VIEWPORTS));

[6] CLAIM: ** One rung-0 "capture" is up to six full browser page-loads plus six OCR jobs, so a quota counted in captures understates cost ~6×.
POINTER: capture.ts:699
OPENED TEXT:
files: fileRecords
      .filter((f) => f.filename.endsWith(".png"))
      .map((f) => ({
        filename: f.filename,
        mime_type: "image/png",

[7] CLAIM: ** The capture path enforces no per-IP limit at all today — only the per-account daily count.
POINTER: capture.ts:234
OPENED TEXT:
// Daily upload limit
  const dailyLimit = await checkDailyUploadLimit(auth.accountId, auth.plan, env);
  if (!dailyLimit.allowed) {
    return apiError(ErrorCodes.RATE_LIMITED, dailyLimit.reason!);
  }

[8] CLAIM: ** The capture path enforces no per-IP limit at all today — only the per-account daily count.
POINTER: items-create.ts:70
OPENED TEXT:
// Rate limit check
  const rateLimit = await checkUploadRateLimit(request, env);
  if (!rateLimit.allowed) {
    return apiError(ErrorCodes.RATE_LIMITED, rateLimit.reason!);
  }

[9] CLAIM: ** Even the wired per-IP limit is inert for MCP-originated work, because the internal Request carries no `CF-Connecting-IP`.
POINTER: mcp.ts:948
OPENED TEXT:
const fakeRequest = new Request("https://internal/api/v1/items", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(reqBody),

[10] CLAIM: ** Even the wired per-IP limit is inert for MCP-originated work, because the internal Request carries no `CF-Connecting-IP`.
POINTER: mcp.ts:953
OPENED TEXT:
});
  // Note: CF-Connecting-IP not forwarded - MCP uploads come from Workers, not the client directly.
  // Unique IP stats rely on browser uploads and direct API calls.

  const ctx = { waitUntil: (p: Promise<any>) => { p.catch(() => {}); }, passThroughOnException: () => {} } as unknown as ExecutionContext;

[11] CLAIM: ** Even the wired per-IP limit is inert for MCP-originated work, because the internal Request carries no `CF-Connecting-IP`.
POINTER: rate-limit.ts:21
OPENED TEXT:
const ip = request.headers.get("CF-Connecting-IP");
  if (!ip) return { allowed: true };

  const ipHash = await hashIp(ip);

[12] CLAIM: ** Three of the six documented ABUSE_BUDGET limits have no enforcement site in the worker, so the doc overstates present coverage.
POINTER: constants.ts:109-113
OPENED TEXT:
/** Max downloads per item before rate-limiting kicks in */
  MAX_DOWNLOADS_PER_ITEM_PER_HOUR: 200,
  /** Max OCR runs per anonymous IP per day */
  MAX_OCR_RUNS_PER_IP_PER_DAY: 100,
  /** Max bandwidth per anonymous IP per hour (500 MiB) */
  MAX_BANDWIDTH_PER_IP_PER_HOUR: 500 * 1024 * 1024,
  /** Max uploads per anonymous IP per hour */
  MAX_UPLOADS_PER_IP_PER_HOUR: 20,
  /** Max items in processing state per IP (prevents queue flooding) */

[13] CLAIM: ** Three of the six documented ABUSE_BUDGET limits have no enforcement site in the worker, so the doc overstates present coverage.
POINTER: ABUSE_PREVENTION.md:18-20
OPENED TEXT:
| Max free upload size | 25 MiB | items-create.ts |
| Max downloads/item/hour | 200 | rate limiting middleware |
| Max OCR runs/IP/day | 100 | queue consumer |
| Max bandwidth/IP/hour | 500 MiB | rate limiting middleware |
| Max uploads/IP/hour | 20 | items-create.ts |
| Max concurrent processing/IP | 5 | items-create.ts |

[14] CLAIM: ** A zero-PII trial account cannot be inserted as-is; `accounts.email` is `NOT NULL UNIQUE`, so rung 0 needs a synthetic per-account sentinel (e.g. `trial+<id>@invalid.local`), and claim is then a plain `UPDATE accounts SET email=?` on the same row.
POINTER: 0001_initial_schema.sql:9
OPENED TEXT:
id            TEXT PRIMARY KEY,
  email         TEXT NOT NULL UNIQUE,
  created_at    TEXT NOT NULL,
  stripe_customer_id TEXT,
  plan          TEXT NOT NULL DEFAULT 'free',

[15] CLAIM: ** A zero-PII trial account cannot be inserted as-is; `accounts.email` is `NOT NULL UNIQUE`, so rung 0 needs a synthetic per-account sentinel (e.g. `trial+<id>@invalid.local`), and claim is then a plain `UPDATE accounts SET email=?` on the same row.
POINTER: auth-verify.ts:39-51
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
  const sessionTokenRaw = generateSessionToken();

[16] CLAIM: ** Setting `plan='trial'` without adding `PLAN_LIMITS.trial` grants 50/day, not 10 — the lookup falls back to free.
POINTER: rate-limit.ts:70
OPENED TEXT:
): Promise<RateLimitResult> {
  const limits = PLAN_LIMITS[plan] ?? PLAN_LIMITS.free;
  const startOfDay = new Date();
  startOfDay.setUTCHours(0, 0, 0, 0);

[17] CLAIM: ** Setting `plan='trial'` without adding `PLAN_LIMITS.trial` grants 50/day, not 10 — the lookup falls back to free.
POINTER: constants.ts:44
OPENED TEXT:
free: {
    uploads_per_day: 50,
    max_total_bytes_per_item: 25 * 1024 * 1024, // 25 MiB
    max_file_bytes: 25 * 1024 * 1024, // 25 MiB
    ttl_hours_max: 24,

[18] CLAIM: ** Rung 0's credential must ride the existing `Mcp-Session-Id` header path and never appear in tool text; enrollment today fires only for `api_key` auth, so this is a real code change, not a config flip.
POINTER: mcp.ts:335
OPENED TEXT:
// and this is an initialize request, create a device and assign Mcp-Session-Id
  if (auth.type === "api_key" && isInitializeRequest(body)) {
    try {
      // Enforce device quota per API key (max 10) — atomic via conditional INSERT
      const sessionToken = generateMcpSessionToken();

[19] CLAIM: ** Rung 0's credential must ride the existing `Mcp-Session-Id` header path and never appear in tool text; enrollment today fires only for `api_key` auth, so this is a real code change, not a config flip.
POINTER: mcp.ts:386
OPENED TEXT:
const headers = new Headers(response.headers);
      headers.set("Mcp-Session-Id", sessionToken);
      return new Response(response.body, { status: response.status, headers });
    } catch (err) {
      log(env, {

[20] CLAIM: ** Rung 0's credential must ride the existing `Mcp-Session-Id` header path and never appear in tool text; enrollment today fires only for `api_key` auth, so this is a real code change, not a config flip.
POINTER: mcp.ts:291-293
OPENED TEXT:
if (!auth) {
    return jsonRpcError(null, -32000, "Authentication required. Pass your API key in the X-API-Key header.");
  }

  // Fix 4: per-account rate limiting
  if (!checkMcpRateLimit(auth.accountId)) {

[21] CLAIM: ** TTL and one-time use defeat hijack direction (a) but not (b); only a code displayed in the user's own CLI and typed on the claim page defeats attacker-generated links.
POINTER: auth-verify.ts:21-23
OPENED TEXT:
// where two concurrent requests could both read used_at IS NULL before either writes.
  const result = await env.DB.prepare(
    "UPDATE magic_tokens SET used_at = ? WHERE token_hash = ? AND used_at IS NULL AND expires_at > ?"
  ).bind(now, tokenHash, now).run();

  if (!result.meta.changes || result.meta.changes === 0) {
    return apiError(ErrorCodes.UNAUTHORIZED, "Invalid, expired, or already used link.");

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }