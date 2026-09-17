You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: MCP device enrollment is already automatic — an API key presented on `initialize` mints a 90-day `Mcp-Session-Id` and an mcp_devices row, so `whipnode init` needs no new device-auth mechanism, only a way to obtain the first key.
POINTER: apps/worker/src/routes/mcp.ts:335
OPENED TEXT:
// and this is an initialize request, create a device and assign Mcp-Session-Id
  if (auth.type === "api_key" && isInitializeRequest(body)) {
    try {
      // Enforce device quota per API key (max 10) — atomic via conditional INSERT
      const sessionToken = generateMcpSessionToken();

[2] CLAIM: The only genuinely new server component is an init-code table in D1, because magic-link verification's sole terminal action is a 302 + cookie to the dashboard, and there is no KV binding to hold a pending code.
POINTER: apps/worker/src/routes/auth-verify.ts:72-77
OPENED TEXT:
// Redirect to dashboard with session cookie
  const headers = new Headers({
    Location: `${env.APP_BASE_URL}/dashboard`,
    "Set-Cookie": `wn_session=${sessionTokenRaw}; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=${30 * 24 * 3600}`,
  });

  return new Response(null, { status: 302, headers });
}

[3] CLAIM: The only genuinely new server component is an init-code table in D1, because magic-link verification's sole terminal action is a 302 + cookie to the dashboard, and there is no KV binding to hold a pending code.
POINTER: apps/worker/src/env.ts:3-7
OPENED TEXT:
// Bindings
  DB: D1Database;
  R2: R2Bucket;
  QUEUE: Queue;
  BROWSER: any;
  AI: any;

  // Vars
  APP_BASE_URL: string;

[4] CLAIM: Magic-link rate limiting is per-email only, so today's signup surface offers no resistance to one-account-per-disposable-inbox farming, even though ip_hash is captured.
POINTER: apps/worker/src/routes/auth-magic-link.ts:26
OPENED TEXT:
const recentTokens = await env.DB.prepare(
    "SELECT COUNT(*) as cnt FROM magic_tokens WHERE email = ? AND created_at > ?"
  )
    .bind(email, fifteenMinAgo)
    .first<{ cnt: number }>();

[5] CLAIM: There is no bot challenge (Turnstile/captcha) anywhere in the tree, so adding one to the init path is net-new work, not configuration.
POINTER: apps/web/app/routes/login.tsx:72-89
OPENED TEXT:
<form className="login-form" onSubmit={handleSubmit}>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="<email>"
          required
          autoFocus
          className="login-email-input"
        />
        <button
          className="btn btn-primary"
          type="submit"
          disabled={loading || !email.trim()}
        >
          {loading ? "Sending..." : "Send sign-in link"}
        </button>
      </form>

      {error && <p className="error-message">{error}</p>}

[6] CLAIM: Free accounts cannot have per-device API keys; segregation must be per-device rows under a single key (free = 1 key × ≤10 devices).
POINTER: packages/protocol/src/constants.ts:48
OPENED TEXT:
ttl_hours_max: 24,
    api_access: true,
    max_api_keys: 1,
    ocr_included: true,
    history: false,

[7] CLAIM: Free accounts cannot have per-device API keys; segregation must be per-device rows under a single key (free = 1 key × ≤10 devices).
POINTER: apps/worker/src/routes/mcp.ts:352
OPENED TEXT:
SELECT ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, 'active'
         WHERE (SELECT COUNT(*) FROM mcp_devices WHERE api_key_id = ? AND status = 'active') < 10`
      )
        .bind(
          deviceId,

[8] CLAIM: The abuse-cost asymmetry sits on captures, not uploads — each capture spawns a Puppeteer browser plus R2 writes plus a queue/OCR job, yet free is metered only by the shared 50 uploads/day counter.
POINTER: apps/worker/src/routes/capture.ts:565
OPENED TEXT:
try {
    const browser = await puppeteer.launch(env.BROWSER);

    for (let i = 0; i < viewportNames.length; i++) {
      const vpName = viewportNames[i];

[9] CLAIM: The abuse-cost asymmetry sits on captures, not uploads — each capture spawns a Puppeteer browser plus R2 writes plus a queue/OCR job, yet free is metered only by the shared 50 uploads/day counter.
POINTER: constants.ts:44
OPENED TEXT:
free: {
    uploads_per_day: 50,
    max_total_bytes_per_item: 25 * 1024 * 1024, // 25 MiB
    max_file_bytes: 25 * 1024 * 1024, // 25 MiB
    ttl_hours_max: 24,

[10] CLAIM: The pricing page already misstates Pro limits relative to code, so the upsell copy is dishonest-by-staleness before any funnel is built.
POINTER: apps/web/app/routes/pricing.tsx:47
OPENED TEXT:
<ul>
            <li>50 MB per file, 100 MB per item</li>
            <li>Higher upload limits</li>
            <li>Extended retention</li>
            <li>Priority processing</li>

[11] CLAIM: The pricing page already misstates Pro limits relative to code, so the upsell copy is dishonest-by-staleness before any funnel is built.
POINTER: packages/protocol/src/constants.ts:55
OPENED TEXT:
uploads_per_day: 1000,
    max_total_bytes_per_item: 250 * 1024 * 1024, // 250 MiB
    max_file_bytes: 50 * 1024 * 1024, // 50 MiB per file
    ttl_hours_max: 168,
    api_access: true,

[12] CLAIM: A no-dashboard upsell surface already has a home — whipnode_status prints plan and today's count but never prints the remaining budget or the wall being approached.
POINTER: apps/worker/src/routes/mcp.ts:673-674
OPENED TEXT:
lines.push(`WhipNode Status`);
  lines.push(`Account: ${account?.email ?? "unknown"} (${account?.plan ?? "free"} plan)`);
  lines.push(`Uploads today: ${uploadCount?.cnt ?? 0}`);
  lines.push("");

  // Group devices

[13] CLAIM: A no-dashboard upsell surface already has a home — whipnode_status prints plan and today's count but never prints the remaining budget or the wall being approached.
POINTER: rate-limit.ts:85
OPENED TEXT:
allowed: false,
      reason: `Daily upload limit reached. Your ${plan} plan allows ${limits.uploads_per_day} uploads per day.`,
    };
  }

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }