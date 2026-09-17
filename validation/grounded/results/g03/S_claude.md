## Framing

Plausible framings: (a) fidelity audit against the two USER-FACT decisions; (b) migration/buildability audit against the tree; (c) attacker-walk of the claim protocol; (d) "is this safe to hand to one unsupervised agent." I pick **(d)**, because the stated constraint is "one implementing agent, no human review between spec and build," and because the spec already concedes its main protocol-level residual risk itself (`ONBOARDING_SPEC.md:32-34`), so unresolved forks and unnamed migrations dominate over further crypto-design debate.

---

```
CLAIM: The spec's only stated schema change for rung 0 - "migration makes email
nullable" - is not expressible as an ALTER TABLE on SQLite/D1; it needs a full
12-step table rebuild of `accounts` (5 referencing FKs, 2 indexes), and the spec
names no rebuild, no index re-creation, no FK deferral.
EVIDENCE: ONBOARDING_SPEC.md:85-87 "migration makes email nullable; SQLite UNIQUE
permits multiple NULLs" vs 0001_initial_schema.sql:9 `email TEXT NOT NULL UNIQUE`;
every migration in the tree is ADD COLUMN only (grep `ALTER TABLE` over
apps/worker/migrations returns 13 ADD COLUMN, 0 RENAME/rebuild). EXTERNAL: SQLite
ALTER TABLE supports only RENAME/ADD COLUMN/DROP COLUMN/RENAME COLUMN
(sqlite.org/lang_altertable.html); D1 rejects `PRAGMA foreign_keys=off` and offers
only `PRAGMA defer_foreign_keys` (developers.cloudflare.com/d1/sql-api/foreign-keys).
CONFIDENCE: 0.85
FALSIFIER: A D1 migration that drops NOT NULL from accounts.email without a table
rebuild, applying cleanly via `wrangler d1 migrations apply`.
```

```
CLAIM: The `npx whipnode` CLI - sole delivery channel for the rung-0 key, the 0600
file, the TTY code, and `init`/`claim`/`status` - does not exist anywhere in the
monorepo, and the spec never names creating/publishing it as work.
EVIDENCE: the only packages are apps/{web,worker} and packages/{renderers,protocol,
catalyst-client,shared}; no package.json declares a `bin`. `npx whipnode` appears
only in ONBOARDING_SPEC.md and docs/panel-review/. Spec lists it as inventory, not
work: ONBOARDING_SPEC.md:251-254.
CONFIDENCE: 0.9
FALSIFIER: A CLI package (bin entry or `apps/cli`) present in the tree.
```

```
CLAIM: Issuance is gated on `initialize` being "mediated by the npx whipnode CLI
wrapper," but the spec names no signal the worker can use to detect wrapper
mediation, so the implementer must invent one.
EVIDENCE: ONBOARDING_SPEC.md:60-61 and 71-73 ("Bare HTTP-MCP clients without the
wrapper get the existing -32000") - no header, UA, or handshake field is named;
mcp.ts:291-293 branches only on `!auth`.
CONFIDENCE: 0.85
FALSIFIER: A line in the spec naming the request field that distinguishes wrapper
from bare client.
```

```
CLAIM: The Authorize step has no defined credential: the endpoint inventory calls
approve "session-authed," but both flows explicitly create no account before
Authorize, and `sessions.account_id` is NOT NULL with an FK to accounts.
EVIDENCE: ONBOARDING_SPEC.md:247 "`POST /auth/device/approve` (session-authed,
kind-aware)" vs :169 "create no account, redirect to the approve page" and :197-199
"creates the account only at Authorize"; 0002_auth_sessions.sql:21,29
`account_id TEXT NOT NULL, FOREIGN KEY (account_id) REFERENCES accounts(id)`.
CONFIDENCE: 0.8
FALSIFIER: Spec text defining a non-session credential (e.g. a claim-scoped cookie)
for approve, or a sessions row that is legal with no account.
```

```
CLAIM: `magic_tokens` has no purpose/binding column, and the spec's migration list
never adds one - so neither the "claim-specific token purpose" nor the
token-to-claim_id binding has storage, and until it ships the existing
/auth/verify will happily consume a claim token and create the merged account the
spec forbids.
EVIDENCE: 0002_auth_sessions.sql:5-13 (columns: id, email, token_hash, created_at,
expires_at, used_at, ip_hash); ONBOARDING_SPEC.md:166-170 requires the purpose;
auth-verify.ts:39-51 upserts an account by email; the spec's named migrations are
only device_codes (:210), trial_counters (:108), email-nullable (:85).
CONFIDENCE: 0.85
FALSIFIER: A `purpose` (or device_code_id) column on magic_tokens in the tree, or
spec text naming that migration.
```

```
CLAIM: The per-IP "1 trial / 24h" cap and the claimed "pre-claim accounts store:
hashed IP" have no column or table anywhere in the spec's migration list or the
schema, leaving the primary per-identity abuse control unimplementable as written.
EVIDENCE: ONBOARDING_SPEC.md:82 "Per-IP-hash issuance cap: 1 trial / 24h (reuse
`hashIp`)" and :257 "Pre-claim accounts store: hashed IP"; accounts columns are
0001_initial_schema.sql:7-14 plus 0009_api_keys_override.sql:2 - no ip_hash;
trial_counters (:108) is day-keyed globals only.
CONFIDENCE: 0.8
FALSIFIER: An accounts.ip_hash column or a per-IP issuance table named in the spec
or present in migrations.
```

```
CLAIM: The spend-ceiling increment as written matches zero rows on the first
request of each UTC day (no row exists for the new `day` key and no INSERT is
specified), which by the spec's own rule returns 429 - and since nothing else
creates the row, trial issuance and trial capture stay 429 for the whole day.
EVIDENCE: ONBOARDING_SPEC.md:114-116 `UPDATE trial_counters SET captures =
captures + 1 WHERE day = ? AND captures < ? -- meta.changes == 0 -> 429`, with
:110-111 "append-only (never reset in place; the day key IS the reset)" and no
INSERT/upsert anywhere in the section.
CONFIDENCE: 0.75
FALSIFIER: Spec text specifying `INSERT OR IGNORE INTO trial_counters(day) VALUES(?)`
(or an upsert) before the conditional UPDATE.
```

```
CLAIM: The 7-day unclaimed-expiry cleanup task names only key revocation and
"delete the account row," with no handling of the child rows that outlive the
trial - items are tombstoned, never deleted - so the DELETE hits FK constraints
(or orphans rows) on every trial account that ever captured anything.
EVIDENCE: ONBOARDING_SPEC.md:100 "revoke the key ... then delete the account row";
cleanup.ts:189 `UPDATE items SET status = 'deleted' WHERE id = ?` (row retained as
"tombstone"); FKs to accounts at 0001_initial_schema.sql:32,57 and
0006_mcp_devices.sql:15-16. EXTERNAL: D1 enforces FK constraints by default
(developers.cloudflare.com/d1/sql-api/foreign-keys).
CONFIDENCE: 0.75
FALSIFIER: A D1 run showing `DELETE FROM accounts` succeeding with live api_keys /
mcp_devices / items rows referencing it, or spec text naming the child deletions.
```

---

**VERDICT INPUT**

- **Q1 Fidelity:** Mostly faithful, and the three USER-FACT owner-calls are all still marked OWNER-CALL (`:94`, `:170`, `:202-204`). But it adds beyond both decisions: bot-score + ASN deny-list infrastructure it admits does not exist (`:79-81`), a global spend-ceiling table, a 7-day expiry task, and a free-tier change shipped as a "default" - devices "3/account" replacing today's hardcoded 10-per-key at `mcp.ts:352`, asserting "change via constants, not code" when the quota is a SQL literal and the scope changes from per-key to per-account (`:240-241`).
- **Q2 Buildability:** Line/mechanism citations to existing code check out (capture.ts:233-237, rate-limit.ts:20-21/:70, mcp.ts:291-293/333-367/~1008, auth-verify.ts:22/38-77, api-keys.ts:139-142, types.ts:32). What is missing is everything new: the email-nullable rebuild, magic_tokens.purpose, per-IP issuance storage, child-row cleanup, the CLI package, and the approve-step credential.
- **Q3 Security:** The protocol's own reasoning is sound (conversion on the key-authed poll, typed code, never-merge) and it states the 0600-theft residual honestly at `:32-34`; the live hole is not the design but the unshipped purpose column, which leaves claim tokens redeemable at `auth-verify.ts:39-51`.
- **Q4 Traps:** Day-rollover counter, approve auth, wrapper detection, and the unresolved raw-key-vs-device-token fork (`:202-204`) that determines rung 0's delivery mechanism and init's simultaneously.

**PUBLISH VERDICT: blocker** — with no human review between spec and build, at least four required schema/cleanup changes are unnamed and the key-delivery fork is unresolved, so a single agent would have to invent the migration plan and the credential model itself.
