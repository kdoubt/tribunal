You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: ** The required schema changes lack a complete migration plan for the existing database.
POINTER: M/0001_initial_schema.sql:9
OPENED TEXT:
id            TEXT PRIMARY KEY,
  email         TEXT NOT NULL UNIQUE,
  created_at    TEXT NOT NULL,
  stripe_customer_id TEXT,
  plan          TEXT NOT NULL DEFAULT 'free',

[2] CLAIM: ** The required schema changes lack a complete migration plan for the existing database.
POINTER: M/0009_api_keys_override.sql:2
OPENED TEXT:
-- Per-account override for max API keys (NULL = use plan default)
ALTER TABLE accounts ADD COLUMN max_api_keys_override INTEGER DEFAULT NULL;

[3] CLAIM: ** The required schema changes lack a complete migration plan for the existing database.
POINTER: M/0002_auth_sessions.sql:5-13
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

[4] CLAIM: ** The specified seven-day account deletion fails against existing foreign keys, and the cleanup lifecycle is incomplete.
POINTER: W/routes/api-keys.ts:136-143
OPENED TEXT:
// Step 1: Revoke — deactivate but keep visible
    await env.DB.prepare("UPDATE api_keys SET status = 'revoked' WHERE id = ?")
      .bind(keyId)
      .run();
    // Cascade: revoke all MCP devices enrolled by this key
    try {
      await env.DB.prepare("UPDATE mcp_devices SET status = 'revoked' WHERE api_key_id = ? AND status = 'active'")
        .bind(keyId)
        .run();
    } catch (err) {
      console.error(`Failed to cascade-revoke devices for key ${keyId}:`, err);
    }

[5] CLAIM: ** The specified seven-day account deletion fails against existing foreign keys, and the cleanup lifecycle is incomplete.
POINTER: M/0001_initial_schema.sql:32
OPENED TEXT:
FOREIGN KEY (account_id) REFERENCES accounts(id)
);

CREATE INDEX idx_api_keys_hash ON api_keys(key_hash);

[6] CLAIM: ** The specified seven-day account deletion fails against existing foreign keys, and the cleanup lifecycle is incomplete.
POINTER: M/0006_mcp_devices.sql:15-16
OPENED TEXT:
FOREIGN KEY (account_id) REFERENCES accounts(id),
  FOREIGN KEY (api_key_id) REFERENCES api_keys(id)
);

CREATE INDEX idx_mcp_devices_session ON mcp_devices(session_token_hash);

[7] CLAIM: ** The specified seven-day account deletion fails against existing foreign keys, and the cleanup lifecycle is incomplete.
POINTER: W/scheduled/cleanup.ts:188-191
OPENED TEXT:
// Mark item as deleted (tombstone for analytics/abuse tracking)
  await env.DB.prepare("UPDATE items SET status = 'deleted' WHERE id = ?")
    .bind(itemId)
    .run();
}

[8] CLAIM: ** First-time init cannot reach the specified session-authenticated Authorize endpoint using the browser authentication mechanism described.
POINTER: M/0002_auth_sessions.sql:21
OPENED TEXT:
id          TEXT PRIMARY KEY,
  account_id  TEXT NOT NULL,
  token_hash  TEXT NOT NULL UNIQUE,
  created_at  TEXT NOT NULL,
  expires_at  TEXT NOT NULL,

[9] CLAIM: ** First-time init cannot reach the specified session-authenticated Authorize endpoint using the browser authentication mechanism described.
POINTER: W/routes/auth-verify.ts:64-74
OPENED TEXT:
// renaming would require a migration for a non-functional change.
  await env.DB.prepare(
    `INSERT INTO sessions (id, account_id, token_hash, created_at, expires_at, last_seen, ip_hash, user_agent)
     VALUES (?, ?, ?, ?, ?, ?, ?, ?)`
  )
    .bind(generateId(), account.id, sessionHash, now, sessionExpires, now, ipHash, await hashUserAgent(ua))
    .run();

  // Redirect to dashboard with session cookie
  const headers = new Headers({
    Location: `${env.APP_BASE_URL}/dashboard`,
    "Set-Cookie": `wn_session=${sessionTokenRaw}; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=${30 * 24 * 3600}`,
  });

  return new Response(null, { status: 302, headers });

[10] CLAIM: ** Credential-file theft can retain authority after conversion, so the stated 24-hour blast-radius rationale does not bound continuing account access.
POINTER: W/routes/account-items.ts:19-20
OPENED TEXT:
// Build dynamic WHERE from query params
  const conditions = ["account_id = ?", "status NOT IN ('deleted')"];
  const binds: (string | number)[] = [auth.accountId];

  const category = url.searchParams.get("category");
  if (category) { conditions.push("category = ?"); binds.push(category); }

[11] CLAIM: ** The enforcement inventory leaves an authenticated Browser-rendering path outside the trial spend and viewport controls.
POINTER: W/routes/router.ts:154-155
OPENED TEXT:
// ── Responsive render ────────────────────────────────
    else if (method === "POST" && path === "/api/v1/render") {
      response = await handleRender(request, env, ctx, auth);
    }
    // ── Raw upload (streaming, for large files) ────────────
    else if (method === "POST" && path === "/api/v1/uploads") {

[12] CLAIM: ** The enforcement inventory leaves an authenticated Browser-rendering path outside the trial spend and viewport controls.
POINTER: W/routes/render.ts:33-35
OPENED TEXT:
): Promise<Response> {
  if (!auth) {
    return apiError(ErrorCodes.UNAUTHORIZED, "Authentication required.");
  }

  let body: {
    item_id?: string;

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }