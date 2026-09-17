A_2seat = union of two isolated Round 0 memos, no ledger, no Round 1

=== MEMO 1 ===

I read the tree before committing. **Alternative framings considered:** (a) growth/funnel question — does rung 0 convert; (b) abuse-economics + auth-plumbing question grounded in what this code actually enforces; (c) consent/PII question. I pick **(b)**, because the artifact settles it decisively: the per-IP controls the proposal leans on are declared but not wired into the capture path, so "what quota" is downstream of "there is no enforcement point yet."

---

**CLAIM 1:** One rung-0 "capture" is up to six full browser page-loads plus six OCR jobs, so a quota counted in captures understates cost ~6×.
**EVIDENCE:** `capture.ts:546` `if (viewportNames.length > 6)`; loop `capture.ts:567-619` opens `browser.newPage()` per viewport; `capture.ts:25` `const MAX_WAIT_MS = 25_000;`; `capture.ts:579` `await new Promise((r) => setTimeout(r, 3000));`; `4k: {width:3840,height:2160}` `capture.ts:20`; queue message filters `.png` files `capture.ts:699`.
**CONFIDENCE:** 0.92
**FALSIFIER:** A per-request browser-time or viewport cap upstream of `handleCapture` that I missed.

**CLAIM 2:** The capture path enforces no per-IP limit at all today — only the per-account daily count.
**EVIDENCE:** `capture.ts:234` calls only `checkDailyUploadLimit`; `checkUploadRateLimit` has exactly one call site, `items-create.ts:70`.
**CONFIDENCE:** 0.9
**FALSIFIER:** An IP check in `index.ts` or `router.ts` ahead of `/api/v1/captures`.

**CLAIM 3:** Even the wired per-IP limit is inert for MCP-originated work, because the internal Request carries no `CF-Connecting-IP`.
**EVIDENCE:** `mcp.ts:948` `new Request("https://internal/api/v1/items", {headers:{"Content-Type":"application/json"}})`; `mcp.ts:953` "CF-Connecting-IP not forwarded"; `rate-limit.ts:21` `if (!ip) return { allowed: true };`
**CONFIDENCE:** 0.88
**FALSIFIER:** Header injection into `fakeRequest` elsewhere in the call chain.

**CLAIM 4:** Three of the six documented ABUSE_BUDGET limits have no enforcement site in the worker, so the doc overstates present coverage.
**EVIDENCE:** `MAX_OCR_RUNS_PER_IP_PER_DAY`, `MAX_BANDWIDTH_PER_IP_PER_HOUR`, `MAX_DOWNLOADS_PER_ITEM_PER_HOUR` appear only at `constants.ts:109-113`; `ABUSE_PREVENTION.md:18-20` lists them as "Enforcement Point: queue consumer / rate limiting middleware".
**CONFIDENCE:** 0.85
**FALSIFIER:** Enforcement by literal value rather than the named constant.

**CLAIM 5:** A zero-PII trial account cannot be inserted as-is; `accounts.email` is `NOT NULL UNIQUE`, so rung 0 needs a synthetic per-account sentinel (e.g. `trial+<id>@invalid.local`), and claim is then a plain `UPDATE accounts SET email=?` on the same row.
**EVIDENCE:** `0001_initial_schema.sql:9` `email TEXT NOT NULL UNIQUE`; upsert-by-email pattern at `auth-verify.ts:39-51`.
**CONFIDENCE:** 0.93
**FALSIFIER:** A later migration relaxing the constraint (I checked 0002–0012 filenames; none is an accounts-email migration).

**CLAIM 6:** Setting `plan='trial'` without adding `PLAN_LIMITS.trial` grants 50/day, not 10 — the lookup falls back to free.
**EVIDENCE:** `rate-limit.ts:70` `const limits = PLAN_LIMITS[plan] ?? PLAN_LIMITS.free;` with `free.uploads_per_day: 50` (`constants.ts:44`).
**CONFIDENCE:** 0.93
**FALSIFIER:** A DB CHECK constraint rejecting unknown plan values.

**CLAIM 7:** Rung 0's credential must ride the existing `Mcp-Session-Id` header path and never appear in tool text; enrollment today fires only for `api_key` auth, so this is a real code change, not a config flip.
**EVIDENCE:** `mcp.ts:335` `if (auth.type === "api_key" && isInitializeRequest(body))`; `mcp.ts:386` `headers.set("Mcp-Session-Id", sessionToken);`; `mcp.ts:291-293` returns "Authentication required" for null auth.
**CONFIDENCE:** 0.9
**FALSIFIER:** An anonymous branch in `handleMcp` I misread.

**CLAIM 8:** TTL and one-time use defeat hijack direction (a) but not (b); only a code displayed in the user's own CLI and typed on the claim page defeats attacker-generated links.
**EVIDENCE:** `auth-verify.ts:21-23` `UPDATE magic_tokens SET used_at=? WHERE token_hash=? AND used_at IS NULL AND expires_at > ?` — single-use is already atomic, yet nothing binds the consumer to the issuing device. ASSUMPTION for the (b) attack path itself (no claim-flow code exists in the artifact).
**CONFIDENCE:** 0.8
**FALSIFIER:** A design where the page never accepts email input, making (b) a no-op.

---

## LADDER SPEC

1. MCP `initialize` with no `X-API-Key`: check `FEATURE_RUNG0_TRIAL` var and a global `trials_issued_today` counter in D1.
2. Gate issuance on hashed `CF-Connecting-IP` from the *outer* request (`mcp.ts:341` already reads it): ≤2 trials/IP/24h, ≤1 per (ip_hash, ua_hash) pair.
3. Insert `accounts` row with sentinel email, `plan='trial'`; add `PLAN_LIMITS.trial` (`uploads_per_day: 10`, `ttl_hours_max: 24`, `max_api_keys: 0`, `history: false`).
4. Insert one `api_keys` row + one `mcp_devices` row; return the session token **only** via the `Mcp-Session-Id` response header (`mcp.ts:386`). Never in `content[].text`.
5. Add `checkUploadRateLimit`-equivalent to `capture.ts` before line 234, with the real IP threaded through from `handleMcp`.
6. Cap trial captures at 1 viewport and `full_page:false`; reject `viewports[]` and `4k` for `plan==='trial'`.
7. Wall hit (daily cap, expired item, `whipnode_render`): worker mints a `claim_tokens` row (nonce hash, account_id, 15 min TTL, single-use) plus a 6-char human code.
8. Error text returns the claim URL **and** the code; the code is the possession proof, the URL alone is inert.
9. User opens link → page shows: "Trial started <ago>, N captures, most recent domain" + a code field + email field.
10. Correct code + email → `UPDATE accounts SET email=?, plan='free'`; magic link sent to that email; account stays `trial` until the magic link is verified.
11. Magic-link verify (`auth-verify.ts`) flips plan to `free` and creates the session. Same `account_id`, same key, same `mcp_devices` rows — no reissue.
12. Kill switch: flip `FEATURE_RUNG0_TRIAL=0` (stops issuance only) and/or set `PLAN_LIMITS.trial.uploads_per_day=0`. Claimed accounts are `plan='free'` and unaffected.

## GUARDRAIL TABLE

| Threat | Control |
|---|---|
| Scripted mass provisioning | ≤2 trials/IP/24h + global daily trial budget (step 1–2); over budget → 401 with install-with-key copy |
| Re-initialize to reset quota | Trial lookup keyed on (ip_hash, ua_hash) before insert; reuse existing account if found |
| 6-viewport / 4k / full_page cost amplification | Trial plan forbids multi-viewport, 4k, full_page (step 6) |
| MCP path bypassing IP limits | Thread real `CF-Connecting-IP` into `handleCapture`; fail closed when absent for `plan==='trial'` |
| Claim link leaked in transcript (hijack a) | Link is inert without the CLI-displayed code; 15 min TTL; single-use via `used_at` UPDATE |
| Attacker-authored claim link (hijack b) | Page shows trial's recent-activity summary; code must match the victim's *own* CLI; email confirmed by magic link before plan flips |
| Trial used as free CDN | `ttl_hours_max: 24`, `history: false`; existing cleanup at `HARD_DELETE_MINUTES_AFTER_EXPIRY` |
| Runaway spend despite caps | Global daily trial budget is a hard ceiling on worst-case browser-seconds; owner can zero it without a deploy |

## VERDICT INPUT

**Q1:** Survivable *only after* code changes — the capture path has no per-IP gate (C2) and the MCP path strips the IP (C3), so the proposal's implied controls do not exist yet; quota must be viewport-seconds, not captures (C1); gate = IP+UA hash, global daily budget; kill switch = feature var + `PLAN_LIMITS.trial` zeroing, which cannot touch claimed accounts because they are `plan='free'`.

**Q2:** Link alone must be inert. Opaque nonce bound to `account_id`, 15 min single-use (reusing the `magic_tokens` atomic-consume pattern), plus a 6-char code printed in the user's CLI; the code defeats (a), and recent-activity display + magic-link email confirmation defeat (b). No bearer credential is ever printed.

**Q3:** Complements. `init` stays as the "I already have a key/account" path and the only way to get a pasteable `wn_` key. The claim flow reuses `magic_tokens`, the `auth-verify` account upsert, and `mcp_devices` unchanged; migration is an in-place `UPDATE accounts` (C5) — same key, same device rows. Recipe page gains a no-header install line above the keyed one.

=== MEMO 2 ===

Plausible framings: onboarding friction, consent to account creation, or compute abuse plus account-claim security. I choose the third: the owner dissolves email-based consent at rung 0, while requiring bounded costs and transcript-safe claiming (USER-FACT).

Pointers below are relative to `artifact/`; `W` = `apps/worker/src`, `P` = `packages/protocol/src`.

CLAIM: The current capture handler does not establish a bounded total cost or a sub-30-second completion guarantee.
EVIDENCE: `W/routes/capture.ts:25,546-547,565-621`: six sequential viewports, each allowing 25-second navigation, plus three seconds for heavy SPAs; screenshots permit `fullPage`, and browser closure precedes the catch rather than occurring in `finally`; `capture.ts:695-706` queues images for processing.
CONFIDENCE: 0.98
FALSIFIER: An encompassing enforced deadline, resource cap, and failure-path browser termination covering this handler.

CLAIM: Existing quotas cannot safely serve as trial capture admission controls.
EVIDENCE: `W/middleware/rate-limit.ts:74-78` counts existing items; `W/routes/capture.ts:234,565,663` checks before launch but inserts afterward; `W/routes/mcp.ts:319-320` executes batches concurrently, and `1009-1013` constructs captures without client-IP headers.
CONFIDENCE: 0.98
FALSIFIER: A shared atomic reservation before every capture launch, including MCP batches and failed attempts.

CLAIM: No stable position on “abuse cost exceeds abuse value”; a capped experiment is defensible, unrestricted renewable trial credentials are not.
EVIDENCE: ASSUMPTION: attacker acquisition costs and capture value are unknown; `P/constants.ts:105-117` declares 20 uploads/IP/hour, five processing items/IP, 100 OCR runs/IP/day and 500 MiB/IP/hour, but `W/middleware/rate-limit.ts:16-58` implements only the first two as item-count checks.
CONFIDENCE: 0.85
FALSIFIER: Measured attacker acquisition costs and fully enforced resource budgets demonstrating the required inequality.

CLAIM: “No email required” is supportable as a redesign, but “zero PII stored” is not established.
EVIDENCE: `apps/worker/migrations/0001_initial_schema.sql:9` requires email; `W/routes/capture.ts:606,613,653-680` stores screenshots, IP/UA hashes and URL-derived notes; `packages/shared/src/hash.ts:20-25` uses deterministic hashes.
CONFIDENCE: 0.97
FALSIFIER: A trial-specific implementation and data audit establishing zero personal information across content, metadata and logs.

CLAIM: A transcript-visible link plus approval by the originating device does not defeat attacker-originated claiming.
EVIDENCE: USER-FACT: the attacker can originate the link and hold its key; ASSUMPTION: approving on that attacker-controlled device adds no victim-device binding; `W/routes/auth-verify.ts:38-50` currently selects or creates accounts by verified email, without such binding.
CONFIDENCE: 0.95
FALSIFIER: A protocol requiring proof from the installation actually connected to the victim’s browser, bound to that same email-verification session.

CLAIM: Claiming free cannot honestly promise recovery of expired content or longer retention.
EVIDENCE: `P/constants.ts:43-51` gives free 50 uploads/day, 24-hour maximum retention and `history: false`; `W/scheduled/cleanup.ts:7-8` specifies deletion of expired objects.
CONFIDENCE: 0.98
FALSIFIER: Changed free-tier retention and implemented recovery semantics.

CLAIM: Rung 0 should complement `init`, reusing enrollment and authentication components while adding a separate claim transaction.
EVIDENCE: USER-FACT: adopted device-code/Authorize design; `W/routes/mcp.ts:333-386` enrolls devices; `W/routes/auth-verify.ts:19-23` atomically consumes magic tokens; `apps/web/app/routes/index.tsx:181-184` currently requires an account and API key.
CONFIDENCE: 0.88
FALSIFIER: A demonstrated replacement handling existing accounts, remote installations, recovery and trial shutdown without an explicit setup path.

**LADDER SPEC**

All following choices are **ASSUMPTION—proposed controls**, not implemented guarantees.

1. Install a local MCP adapter generating a signing key outside model context. Register its public key idempotently; bind one trial account/device to it. Never return credentials through tool output. Make account email nullable and add an explicit trial state.

2. Gate issuance using trusted edge IP: one installation per rolling 24 hours per IPv4 or IPv6 /64, initially 100 globally/day. Persist short-lived keyed network counters; disclose this pseudonymous metadata. ASN deny rules supplement caps; bot scores are optional only if available. No proof-of-work dependency.

3. Start with three attempted captures/rolling 24 hours, ten lifetime, seven-day trial validity and 24-hour item expiry. Aggregate network allowances across credentials. Reinstallation grants no network-budget reset.

4. Atomically reserve account, network and global budgets in D1 before work; charge failures. Allow one capture per installation/network and two globally. Enforce across direct API and MCP; deny other expensive trial tools.

5. Permit one 1280×800 viewport, no full-page capture, bounded subrequests/download bytes and screenshot output. Set a 25-second total execution deadline with verified browser termination; disable export fallbacks. Cap downstream OCR/AI work and retries. Target first screenshot under 30 seconds; do not promise arbitrary-site success.

6. Start at 100 attempts/day globally: at most 2,500 browser-seconds **if termination is proven**, plus separately reserved processing/storage costs. Owner sets a monetary ceiling from measured worst-case cost. Default off until failure-path tests validate it.

7. A D1 `trial_enabled=false` blocks issuance and new trial work; claimed accounts, claim completion and existing reads continue. Recheck state at admission.

8. At a wall return structured reason/reset information and `https://whipnode.com/claim/<public-id>`. Store its account/device binding server-side; expire after ten minutes. GET neither authorizes nor reveals activity.

9. The page establishes a private browser session and challenges the local adapter over loopback, with strict Host/Origin checks. Its signature must match the trial public key and browser nonce. Reject remote relaying; unsupported environments use explicit `init`. No codes or secrets travel through transcripts.

10. After possession proof, show device fingerprint, recent capture origins/times, quota changes and retention. Enter email here; verify through a purpose-bound, single-use email flow tied to this browser session. Then one **Authorize** click commits. Email verification remains an additional interaction.

11. Atomically attach email and preserve account/item IDs; replace trial credentials and device session without grace. Carry the proven device row. For an existing email, explicitly approve merging items/device into that account, preserving its key limit and accumulated usage; revoke trial credentials.

12. Reuse the prior approve page and device-code state-machine work, with the stronger binding above. Recipe page offers “Try without email” and `init` for immediate account setup. State quotas, expiry, processing limits and availability; preserve pro upgrade.

Exact relayed copy:

> Trial limit reached: {reason}. {retry_information}  
> Claim this installation: https://whipnode.com/claim/{public_id}  
> Open on the computer running WhipNode. Email verification and Authorize are required. Free includes 50 uploads/day and 24-hour retention. Claiming cannot restore expired captures. This link alone grants no access.

**GUARDRAIL TABLE**

| Threat | Proposed control |
|---|---|
| Credential farming | Network/lifetime caps; global budget |
| Concurrent quota bypass | Atomic pre-work reservations |
| Expensive failures | Deadline, termination, charged attempts |
| Transcript-link theft | Public locator; private local possession proof |
| Attacker-link phishing | Victim-browser/local-key match; session-bound email |
| Credential carryover | Reissue without grace; revoke trial sessions |
| Misleading upsell | Explicit retention and recovery limits |
| Farm overwhelms operations | Independent trial kill switch |

**VERDICT INPUT**

Q1: Reject current economics; permit only a measured, tightly capped pilot.  
Q2: Reject bearer links and device approval alone; require browser-local installation binding.  
Q3: Complement `init`; preserve data identity, replace credentials, and revise recipe promises.
