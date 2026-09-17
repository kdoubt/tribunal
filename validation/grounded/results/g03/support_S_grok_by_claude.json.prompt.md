You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The spec encodes both USER-FACT decisions (init as device-code wrapping magic-link with one Authorize click and key + `mcp_devices` enrollment; rung 0 as anonymous no-PII trial, non-bearer locator, possession-proof same-key conversion, complementary to init) but silently defaults two OWNER-CALLs (trial quota 10, free devices 3) and leaves Turnstile and raw-key vs device-token unresolved for an agent that will not get a human call.
POINTER: ONBOARDING_SPEC.md:94
OPENED TEXT:
|---|---|
| Items (captures+uploads)/day | 10 (OWNER-CALL; codex-seat dissent: 3 until billing telemetry) |
| Viewports per capture | 1; `full_page`, `4k`, `1080p` rejected in capture validation |
| Retention (TTL) | 24h |
| Concurrent processing | <= 2 (reservation/release, not just a constant) |

[2] CLAIM: The required `accounts.email` nullability change has no complete D1 migration: email is `TEXT NOT NULL UNIQUE`, and this repo only ever `ADD COLUMN`s — SQLite/D1 cannot `ALTER` away NOT NULL without a table rebuild the spec does not give.
POINTER: apps/worker/migrations/0001_initial_schema.sql:9
OPENED TEXT:
id            TEXT PRIMARY KEY,
  email         TEXT NOT NULL UNIQUE,
  created_at    TEXT NOT NULL,
  stripe_customer_id TEXT,
  plan          TEXT NOT NULL DEFAULT 'free',

[3] CLAIM: The required `accounts.email` nullability change has no complete D1 migration: email is `TEXT NOT NULL UNIQUE`, and this repo only ever `ADD COLUMN`s — SQLite/D1 cannot `ALTER` away NOT NULL without a table rebuild the spec does not give.
POINTER: ONBOARDING_SPEC.md:85-87
OPENED TEXT:
On pass: create `accounts` row (`plan='trial'`, `email=NULL` - migration
makes email nullable; SQLite UNIQUE permits multiple NULLs, no
sentinels), one `wn_` key, enroll the `mcp_devices` row via the existing
mcp.ts:333-367 path.

### Trial shape (`PLAN_LIMITS.trial` + enforcement inventory)

[4] CLAIM: The required `accounts.email` nullability change has no complete D1 migration: email is `TEXT NOT NULL UNIQUE`, and this repo only ever `ADD COLUMN`s — SQLite/D1 cannot `ALTER` away NOT NULL without a table rebuild the spec does not give.
POINTER: 0009_api_keys_override.sql:2
OPENED TEXT:
-- Per-account override for max API keys (NULL = use plan default)
ALTER TABLE accounts ADD COLUMN max_api_keys_override INTEGER DEFAULT NULL;

[5] CLAIM: Spend-ceiling and token-purpose schema are not implementable as written: the only `trial_counters` SQL is an `UPDATE` whose `meta.changes == 0` is defined as 429, so a missing UTC-day row (every new day, and forever if no INSERT is specified) looks like cap exhaustion; no table is named for the 1/IP/24h cap; `magic_tokens` has no `purpose` column the claim/init verify split requires.
POINTER: ONBOARDING_SPEC.md:108-115
OPENED TEXT:
Table: `trial_counters (day TEXT PRIMARY KEY, issuances INTEGER NOT NULL
DEFAULT 0, captures INTEGER NOT NULL DEFAULT 0)` - UTC-day keyed,
append-only (never reset in place; the day key IS the reset, matching the
00:00 UTC wall copy). Every increment is atomic-conditional immediately
before allocation:

    UPDATE trial_counters SET captures = captures + 1
      WHERE day = ? AND captures < ?    -- meta.changes == 0 -> 429

Start caps ~250 issuances / 200-500 captures per day; alert email at 80%.
Cap-exhaustion 429 of legitimate onboarding is ACCEPTED fail-closed

[6] CLAIM: Spend-ceiling and token-purpose schema are not implementable as written: the only `trial_counters` SQL is an `UPDATE` whose `meta.changes == 0` is defined as 429, so a missing UTC-day row (every new day, and forever if no INSERT is specified) looks like cap exhaustion; no table is named for the 1/IP/24h cap; `magic_tokens` has no `purpose` column the claim/init verify split requires.
POINTER: apps/worker/migrations/0002_auth_sessions.sql:5-13
OPENED TEXT:
-- ── Magic Link Tokens ────────────────────────────────────────────
CREATE TABLE IF NOT EXISTS magic_tokens (
  id          TEXT PRIMARY KEY,
  email       TEXT NOT NULL,
  token_hash  TEXT NOT NULL UNIQUE,
  created_at  TEXT NOT NULL,
  expires_at  TEXT NOT NULL,
  used_at     TEXT,
  ip_hash     TEXT
);

CREATE INDEX idx_magic_tokens_hash ON magic_tokens(token_hash);
CREATE INDEX idx_magic_tokens_email ON magic_tokens(email);

[7] CLAIM: `POST /auth/device/approve (session-authed)` cannot run for claim or init as specified: both verify paths create no account, so no `wn_session` exists, while `sessions.account_id` is `NOT NULL`.
POINTER: ONBOARDING_SPEC.md:246-247
OPENED TEXT:
Worker: trial issuance inside unauthenticated `initialize`;
`POST /auth/device/start` (kind-aware); `POST /auth/device/poll`;
`POST /auth/device/approve` (session-authed, kind-aware); claim/init
magic-token purpose in auth-magic-link/auth-verify; `GET /claim/:id`
page; capture validation for trial options; trial cleanup task.

[8] CLAIM: `POST /auth/device/approve (session-authed)` cannot run for claim or init as specified: both verify paths create no account, so no `wn_session` exists, while `sessions.account_id` is `NOT NULL`.
POINTER: 0002_auth_sessions.sql:21
OPENED TEXT:
id          TEXT PRIMARY KEY,
  account_id  TEXT NOT NULL,
  token_hash  TEXT NOT NULL UNIQUE,
  created_at  TEXT NOT NULL,
  expires_at  TEXT NOT NULL,

[9] CLAIM: `POST /auth/device/approve (session-authed)` cannot run for claim or init as specified: both verify paths create no account, so no `wn_session` exists, while `sessions.account_id` is `NOT NULL`.
POINTER: auth-verify.ts:53-69
OPENED TEXT:
// Create session
  const sessionTokenRaw = generateSessionToken();
  const sessionHash = await hashApiKey(sessionTokenRaw);
  const sessionExpires = new Date(Date.now() + 30 * 24 * 3600_000).toISOString(); // 30 days
  const ip = request.headers.get("CF-Connecting-IP") ?? "";
  const ipHash = ip ? await hashIp(ip) : null;
  const ua = request.headers.get("User-Agent") ?? "";

  // Note: the "user_agent" column stores a SHA-256 hash (via hashUserAgent),
  // not the raw User-Agent string. The column name is a legacy misnomer —
  // renaming would require a migration for a non-functional change.
  await env.DB.prepare(
    `INSERT INTO sessions (id, account_id, token_hash, created_at, expires_at, last_seen, ip_hash, user_agent)
     VALUES (?, ?, ?, ?, ?, ?, ?, ?)`
  )
    .bind(generateId(), account.id, sessionHash, now, sessionExpires, now, ipHash, await hashUserAgent(ua))
    .run();

  // Redirect to dashboard with session cookie
  const headers = new Headers({

[10] CLAIM: The named 7-day unclaimed-trial cleanup will not run on any cron tick where no items are expired, because `handleScheduled` returns before later tasks when that SELECT is empty.
POINTER: ONBOARDING_SPEC.md:100
OPENED TEXT:
| OCR + prompt packs | included (it is the product) |
| Unclaimed account expiry | 7 days - a NEW cleanup.ts task: revoke the key (existing cascade revokes devices, api-keys.ts:139-142), then delete the account row; a PENDING claim freezes this expiry |

Enforcement is not `PLAN_LIMITS` alone: capture-option validation,
account-level device quota, and concurrency reservation each need

[11] CLAIM: The named 7-day unclaimed-trial cleanup will not run on any cron tick where no items are expired, because `handleScheduled` returns before later tasks when that SELECT is empty.
POINTER: apps/worker/src/scheduled/cleanup.ts:14-24
OPENED TEXT:
// Find expired items not yet cleaned up
  const expired = await env.DB.prepare(
    `SELECT id FROM items
     WHERE expires_at <= ? AND status NOT IN ('deleted')
     LIMIT ?`
  )
    .bind(now, CLEANUP_BATCH_SIZE)
    .all<{ id: string }>();

  if (!expired.results || expired.results.length === 0) {
    return;
  }

  for (const item of expired.results) {
    await cleanupItem(item.id, env);

[12] CLAIM: Wrapper-only trial issuance is not a testable worker condition: no request header/UA is named, so an implementer either mints on every unauthenticated `initialize` (raw `wn_` in an unnamed response header — a new bearer surface, plus a burned IP slot) or invents a discriminator; enrollment “via mcp.ts:333-367” also cannot run on the arriving `auth == null` request without an unstated control-flow rewrite.
POINTER: ONBOARDING_SPEC.md:62-73
OPENED TEXT:
Trigger: unauthenticated MCP `initialize` **mediated by the `npx
whipnode` CLI wrapper**, when `FEATURE_ENABLE_ANON_TRIAL=true`. Two
transport realities (panel-verified):

- The worker rejects unauthenticated MCP before parsing today
  (mcp.ts:291-293); issuance is a carve-out for `initialize` only.
- **The raw `wn_` key never travels in JSON-RPC/tool text.** A vanilla
  remote-MCP client has no way to persist an in-band key anyway. The key
  is delivered once in a response header consumed by the `npx whipnode`
  stdio wrapper, which writes the 0600 credential file and thereafter
  supplies `X-API-Key`. Bare HTTP-MCP clients without the wrapper get the
  existing `-32000` plus one line: run `npx whipnode init`.
- Reconnects authenticate with the stored key; they never mint a second
  trial (the 1/IP/24h cap also blocks accidental remints).

[13] CLAIM: Wrapper-only trial issuance is not a testable worker condition: no request header/UA is named, so an implementer either mints on every unauthenticated `initialize` (raw `wn_` in an unnamed response header — a new bearer surface, plus a burned IP slot) or invents a discriminator; enrollment “via mcp.ts:333-367” also cannot run on the arriving `auth == null` request without an unstated control-flow rewrite.
POINTER: mcp.ts:291-293
OPENED TEXT:
if (!auth) {
    return jsonRpcError(null, -32000, "Authentication required. Pass your API key in the X-API-Key header.");
  }

  // Fix 4: per-account rate limiting
  if (!checkMcpRateLimit(auth.accountId)) {

[14] CLAIM: Wrapper-only trial issuance is not a testable worker condition: no request header/UA is named, so an implementer either mints on every unauthenticated `initialize` (raw `wn_` in an unnamed response header — a new bearer surface, plus a burned IP slot) or invents a discriminator; enrollment “via mcp.ts:333-367” also cannot run on the arriving `auth == null` request without an unstated control-flow rewrite.
POINTER: mcp.ts:335
OPENED TEXT:
// and this is an initialize request, create a device and assign Mcp-Session-Id
  if (auth.type === "api_key" && isInitializeRequest(body)) {
    try {
      // Enforce device quota per API key (max 10) — atomic via conditional INSERT
      const sessionToken = generateMcpSessionToken();

[15] CLAIM: Wrapper-only trial issuance is not a testable worker condition: no request header/UA is named, so an implementer either mints on every unauthenticated `initialize` (raw `wn_` in an unnamed response header — a new bearer surface, plus a burned IP slot) or invents a discriminator; enrollment “via mcp.ts:333-367” also cannot run on the arriving `auth == null` request without an unstated control-flow rewrite.
POINTER: env.ts:38-41
OPENED TEXT:
// Feature flags
  FEATURE_ENABLE_ZIP_BUNDLES?: string;
  FEATURE_ENABLE_EMAIL_SHARING?: string;
  FEATURE_ENABLE_FACE_BLOCKING?: string;
}

[16] CLAIM: Stolen 0600 file is already a complete claim credential (claim-start and converting poll both authenticate with the trial key); combining it with a leaked locator adds nothing the spec’s residual-risk paragraph does not already grant, while “a PENDING claim freezes this expiry” with no `expires_at > now` predicate lets that same key refresh freeze indefinitely (`whipnode claim` inside the 10-min TTL).
POINTER: ONBOARDING_SPEC.md:31-33
OPENED TEXT:
locators; conversion requires possession of the trial key. Residual risk,
stated honestly: **theft of the 0600 credential file grants claim
authority over that trial** - blast radius is capped by never-merge, zero
pre-claim identity, and 24h item TTL. If credential-file theft is observed
in practice, the recorded escalation path is the codex-seat dissent: an
Ed25519 installation keypair with loopback signing, so the possession
secret never exists in any file an agent can read.

[17] CLAIM: Two state machines on one `device_codes` table are sketched, not composed: no unique partial index backs “one pending claim” / “concurrent starts can't race two emails”; init Authorize must create the account and later poll may return key material, claim Authorize must not mutate the account and must not return a key; activity snapshot “parsed” from `items.note` is lossy because a client `note` replaces the URL-bearing default and Google Doc captures store the title, not the URL (`captured_url` is response-only).
POINTER: ONBOARDING_SPEC.md:154-156
OPENED TEXT:
key-possession + code-match design. Claim-start authenticates with
   the trial key (key must be `status='active'`); it invalidates any
   prior pending claim for this account (one pending claim per trial;
   concurrent starts can't race two emails).
2. Claim-start snapshots the trial's recent capture hosts
   (`{host, created_at}` x3, parsed at claim time - there is no
   `captured_url` column; items carry the URL only in `note`) onto the

[18] CLAIM: Two state machines on one `device_codes` table are sketched, not composed: no unique partial index backs “one pending claim” / “concurrent starts can't race two emails”; init Authorize must create the account and later poll may return key material, claim Authorize must not mutate the account and must not return a key; activity snapshot “parsed” from `items.note` is lossy because a client `note` replaces the URL-bearing default and Google Doc captures store the title, not the URL (`captured_url` is response-only).
POINTER: capture.ts:658-674
OPENED TEXT:
const defaultNote = viewportNames.length === 1
    ? `Screenshot of ${url} (${viewportNames[0]})`
    : `Screenshots of ${url} (${viewportNames.join(", ")})`;

  // Insert item
  await env.DB.prepare(
    `INSERT INTO items (id, account_id, created_at, expires_at, status, note,
     client_type, origin, file_count, total_bytes, ocr_status, moderation_status,
     public_token, source_ip_hash, user_agent_hash, device_id)
     VALUES (?, ?, ?, ?, 'processing', ?, 'agent', 'capture', ?, ?, 'pending', 'pending', ?, ?, ?, ?)`
  )
    .bind(
      itemId,
      auth.accountId,
      now,
      expiresAt,
      note ?? defaultNote,
      fileRecords.length,
      totalBytes,
      publicToken,

[19] CLAIM: Two state machines on one `device_codes` table are sketched, not composed: no unique partial index backs “one pending claim” / “concurrent starts can't race two emails”; init Authorize must create the account and later poll may return key material, claim Authorize must not mutate the account and must not return a key; activity snapshot “parsed” from `items.note` is lossy because a client `note` replaces the URL-bearing default and Google Doc captures store the title, not the URL (`captured_url` is response-only).
POINTER: capture.ts:494
OPENED TEXT:
txtItemId, auth.accountId, txtNow, txtExpiresAt,
              note ?? `Google Doc: ${docTitle}`,
              txtBytes.byteLength, txtPublicToken, ipHash, uaHash,
              auth.type === "mcp_device" ? auth.deviceId : null
            )

[20] CLAIM: Two state machines on one `device_codes` table are sketched, not composed: no unique partial index backs “one pending claim” / “concurrent starts can't race two emails”; init Authorize must create the account and later poll may return key material, claim Authorize must not mutate the account and must not return a key; activity snapshot “parsed” from `items.note` is lossy because a client `note` replaces the URL-bearing default and Google Doc captures store the title, not the URL (`captured_url` is response-only).
POINTER: capture.ts:712
OPENED TEXT:
status: "processing",
    captured_url: url,
    viewports: viewportNames,
    url: `${base}/p/${itemId}`,
    json_url: `${base}/api/v1/items/${itemId}`,

[21] CLAIM: Two state machines on one `device_codes` table are sketched, not composed: no unique partial index backs “one pending claim” / “concurrent starts can't race two emails”; init Authorize must create the account and later poll may return key material, claim Authorize must not mutate the account and must not return a key; activity snapshot “parsed” from `items.note` is lossy because a client `note` replaces the URL-bearing default and Google Doc captures store the title, not the URL (`captured_url` is response-only).
POINTER: 0001_initial_schema.sql:40-58
OPENED TEXT:
CREATE TABLE IF NOT EXISTS items (
  id                TEXT PRIMARY KEY,
  account_id        TEXT,
  created_at        TEXT NOT NULL,
  expires_at        TEXT NOT NULL,
  status            TEXT NOT NULL DEFAULT 'processing',
  note              TEXT,
  client_type       TEXT NOT NULL,
  origin            TEXT,
  file_count        INTEGER NOT NULL,
  total_bytes       INTEGER NOT NULL,
  ocr_status        TEXT NOT NULL DEFAULT 'pending',
  moderation_status TEXT NOT NULL DEFAULT 'pending',
  public_token      TEXT NOT NULL,
  source_ip_hash    TEXT,
  user_agent_hash   TEXT,

  FOREIGN KEY (account_id) REFERENCES accounts(id)
);

CREATE INDEX idx_items_expires ON items(expires_at)
  WHERE status NOT IN ('deleted');

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }