You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The spec's only stated schema change for rung 0 - "migration makes email
POINTER: ONBOARDING_SPEC.md:85-87
OPENED TEXT:
On pass: create `accounts` row (`plan='trial'`, `email=NULL` - migration
makes email nullable; SQLite UNIQUE permits multiple NULLs, no
sentinels), one `wn_` key, enroll the `mcp_devices` row via the existing
mcp.ts:333-367 path.

### Trial shape (`PLAN_LIMITS.trial` + enforcement inventory)

[2] CLAIM: The spec's only stated schema change for rung 0 - "migration makes email
POINTER: 0001_initial_schema.sql:9
OPENED TEXT:
id            TEXT PRIMARY KEY,
  email         TEXT NOT NULL UNIQUE,
  created_at    TEXT NOT NULL,
  stripe_customer_id TEXT,
  plan          TEXT NOT NULL DEFAULT 'free',

[3] CLAIM: The `npx whipnode` CLI - sole delivery channel for the rung-0 key, the 0600
POINTER: ONBOARDING_SPEC.md:251-254
OPENED TEXT:
CLI (`npx whipnode`, stdio MCP wrapper + subcommands): `init`, `claim`,
`status`; TTY checks (UX guard); 0600 credential file; key printed never
(delivered via header/poll and written straight to file).

## Data + privacy posture

Pre-claim accounts store: hashed IP, derived device name, captured page

[4] CLAIM: Issuance is gated on `initialize` being "mediated by the npx whipnode CLI
POINTER: ONBOARDING_SPEC.md:60-61
OPENED TEXT:
### Issuance (on `initialize` only)

Trigger: unauthenticated MCP `initialize` **mediated by the `npx
whipnode` CLI wrapper**, when `FEATURE_ENABLE_ANON_TRIAL=true`. Two
transport realities (panel-verified):

[5] CLAIM: Issuance is gated on `initialize` being "mediated by the npx whipnode CLI
POINTER: mcp.ts:291-293
OPENED TEXT:
if (!auth) {
    return jsonRpcError(null, -32000, "Authentication required. Pass your API key in the X-API-Key header.");
  }

  // Fix 4: per-account rate limiting
  if (!checkMcpRateLimit(auth.accountId)) {

[6] CLAIM: The Authorize step has no defined credential: the endpoint inventory calls
POINTER: ONBOARDING_SPEC.md:247
OPENED TEXT:
`POST /auth/device/start` (kind-aware); `POST /auth/device/poll`;
`POST /auth/device/approve` (session-authed, kind-aware); claim/init
magic-token purpose in auth-magic-link/auth-verify; `GET /claim/:id`
page; capture validation for trial options; trial cleanup task.

[7] CLAIM: The Authorize step has no defined credential: the endpoint inventory calls
POINTER: 0002_auth_sessions.sql:21
OPENED TEXT:
id          TEXT PRIMARY KEY,
  account_id  TEXT NOT NULL,
  token_hash  TEXT NOT NULL UNIQUE,
  created_at  TEXT NOT NULL,
  expires_at  TEXT NOT NULL,

[8] CLAIM: `magic_tokens` has no purpose/binding column, and the spec's migration list
POINTER: 0002_auth_sessions.sql:5-13
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

[9] CLAIM: `magic_tokens` has no purpose/binding column, and the spec's migration list
POINTER: ONBOARDING_SPEC.md:166-170
OPENED TEXT:
verify handler MUST NOT run for claim tokens - today it upserts an
   account by email and 302s to /dashboard (auth-verify.ts:38-77), which
   would both violate never-merge and strand the flow. Claim-verify
   instead: prove the mailbox, write `verified_email` onto the pairing
   row, create no account, redirect to the approve page.
4. Approve page shows: the snapshot hosts + times ("only authorize if
   these are yours"; empty snapshot renders "0 captures - only approve
   if you just started this trial"), device name + created-at, free-tier
   bullets, and an input where the human **types the 8-char code from

[10] CLAIM: `magic_tokens` has no purpose/binding column, and the spec's migration list
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

[11] CLAIM: The per-IP "1 trial / 24h" cap and the claimed "pre-claim accounts store:
POINTER: ONBOARDING_SPEC.md:82
OPENED TEXT:
list - neither exists in-repo today, both must be built).
- Per-IP-hash issuance cap: 1 trial / 24h (reuse `hashIp`).
- Global issuance counter under cap (see Spend ceiling).

On pass: create `accounts` row (`plan='trial'`, `email=NULL` - migration

[12] CLAIM: The per-IP "1 trial / 24h" cap and the claimed "pre-claim accounts store:
POINTER: 0001_initial_schema.sql:7-14
OPENED TEXT:
CREATE TABLE IF NOT EXISTS accounts (
  id            TEXT PRIMARY KEY,
  email         TEXT NOT NULL UNIQUE,
  created_at    TEXT NOT NULL,
  stripe_customer_id TEXT,
  plan          TEXT NOT NULL DEFAULT 'free',
  status        TEXT NOT NULL DEFAULT 'active'
);

CREATE INDEX idx_accounts_email ON accounts(email);
CREATE INDEX idx_accounts_stripe ON accounts(stripe_customer_id)

[13] CLAIM: The per-IP "1 trial / 24h" cap and the claimed "pre-claim accounts store:
POINTER: 0009_api_keys_override.sql:2
OPENED TEXT:
-- Per-account override for max API keys (NULL = use plan default)
ALTER TABLE accounts ADD COLUMN max_api_keys_override INTEGER DEFAULT NULL;

[14] CLAIM: The spend-ceiling increment as written matches zero rows on the first
POINTER: ONBOARDING_SPEC.md:114-116
OPENED TEXT:
UPDATE trial_counters SET captures = captures + 1
      WHERE day = ? AND captures < ?    -- meta.changes == 0 -> 429

Start caps ~250 issuances / 200-500 captures per day; alert email at 80%.
Cap-exhaustion 429 of legitimate onboarding is ACCEPTED fail-closed
behavior - the ceiling converts worst-case abuse from an unbounded

[15] CLAIM: The 7-day unclaimed-expiry cleanup task names only key revocation and
POINTER: ONBOARDING_SPEC.md:100
OPENED TEXT:
| OCR + prompt packs | included (it is the product) |
| Unclaimed account expiry | 7 days - a NEW cleanup.ts task: revoke the key (existing cascade revokes devices, api-keys.ts:139-142), then delete the account row; a PENDING claim freezes this expiry |

Enforcement is not `PLAN_LIMITS` alone: capture-option validation,
account-level device quota, and concurrency reservation each need

[16] CLAIM: The 7-day unclaimed-expiry cleanup task names only key revocation and
POINTER: cleanup.ts:189
OPENED TEXT:
// Mark item as deleted (tombstone for analytics/abuse tracking)
  await env.DB.prepare("UPDATE items SET status = 'deleted' WHERE id = ?")
    .bind(itemId)
    .run();
}

[17] CLAIM: The 7-day unclaimed-expiry cleanup task names only key revocation and
POINTER: 0001_initial_schema.sql:32
OPENED TEXT:
FOREIGN KEY (account_id) REFERENCES accounts(id)
);

CREATE INDEX idx_api_keys_hash ON api_keys(key_hash);

[18] CLAIM: The 7-day unclaimed-expiry cleanup task names only key revocation and
POINTER: 0006_mcp_devices.sql:15-16
OPENED TEXT:
FOREIGN KEY (account_id) REFERENCES accounts(id),
  FOREIGN KEY (api_key_id) REFERENCES api_keys(id)
);

CREATE INDEX idx_mcp_devices_session ON mcp_devices(session_token_hash);

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }