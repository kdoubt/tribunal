Plausible framings are decision fidelity, implementation readiness, and protocol security. I choose **implementation readiness**, using fidelity and attacker walkthroughs as acceptance checks, because the brief sends this document directly to one implementing agent.

Pointers below are relative to `artifact/`; `S` means `ONBOARDING_SPEC.md`, `W` means `apps/worker/src`, and `M` means `apps/worker/migrations`.

CLAIM: The draft preserves the central ladder but adds product decisions and operationally selects raw trial keys despite calling that choice unresolved.  
EVIDENCE: USER-FACT; S:22 preserves complementary init, S:185–189 preserves same-key conversion; S:68–72 mandates raw-key delivery and `X-API-Key`, contradicting S:202–204; S:79–82 adds mandatory bot-score/ASN gates; S:236–241 introduces additional free-tier owner calls and ships three devices/account. Trial quota and Turnstile remain explicitly open at S:94 and S:163.  
CONFIDENCE: 0.96  
FALSIFIER: A supplied decision authorizes those additions and reconciles mandatory raw-key delivery with the unresolved credential choice.

CLAIM: The required schema changes lack a complete migration plan for the existing database.  
EVIDENCE: S:85–87 says only “migration makes email nullable”; M/0001_initial_schema.sql:9 declares `email TEXT NOT NULL UNIQUE`, :16–18 adds indexes, and M/0009_api_keys_override.sql:2 adds a column that must survive any rebuild. S:164–169 requires token purposes and pairing association, but M/0002_auth_sessions.sql:5–13 contains neither. S:108–115 and :210–220 provide table sketches without an ordered migration plan. EXTERNAL: [D1 foreign-key documentation](https://developers.cloudflare.com/d1/sql-api/foreign-keys/) specifies enforced foreign keys and transaction-scoped deferral, rather than disabling enforcement.  
CONFIDENCE: 0.97  
FALSIFIER: An artifact section supplies executable migrations preserving existing data, columns, indexes, and references while adding token-purpose/pairing bindings and both new tables.

CLAIM: The specified seven-day account deletion fails against existing foreign keys, and the cleanup lifecycle is incomplete.  
EVIDENCE: S:100 prescribes revoking the key and then deleting the account; W/routes/api-keys.ts:136–143 only updates key/device statuses. M/0001_initial_schema.sql:32 and :57 retain account references, and M/0006_mcp_devices.sql:15–16 retains account/key references. W/scheduled/cleanup.ts:188–191 retains item tombstones; :22–24 returns before subsequent maintenance when no items expire. S:100 freezes expiry for pending claims, while S:210–220 supplies expiry fields without specifying a pairing expiry/purge task. EXTERNAL: [D1 foreign-key enforcement](https://developers.cloudflare.com/d1/sql-api/foreign-keys/).  
CONFIDENCE: 0.98  
FALSIFIER: A specified cleanup order removes or detaches dependents, expires/purges pairings, and executes independently of the expired-item batch.

CLAIM: First-time init cannot reach the specified session-authenticated Authorize endpoint using the browser authentication mechanism described.  
EVIDENCE: S:195–198 postpones account creation until Authorize; S:247 requires Authorize to be “session-authed.” M/0002_auth_sessions.sql:21,29 requires every existing session to reference an account; W/routes/auth-verify.ts:64–74 creates that account-bound session cookie. S:164–169 describes mailbox verification and redirect but no temporary browser credential or its binding to the pairing.  
CONFIDENCE: 0.96  
FALSIFIER: A described pre-account browser authentication mechanism binds the verified mailbox, pairing, purpose, and Authorize request without making the locator a bearer credential.

CLAIM: The shared pairing design does not fully specify atomic completion, identity binding, or expiry enforcement, despite naming states and revoked-key checks.  
EVIDENCE: S:185–191 specifies one account UPDATE checking approved/unconsumed pairing and active key/device, but does not specify how pairing consumption commits with conversion or include `expires_at` in its listed assertions. S:210–220 has no key/device identifiers identifying the device whose activity must be checked. S:154–156 promises one pending claim; S:218 also introduces `verified` and `approved` without defining whether replacement invalidates those states.  
CONFIDENCE: 0.91  
FALSIFIER: Concrete transition predicates and transactional operations identify the bound account/key/device, enforce TTL, define replacement for every live state, and handle concurrent polls and retries consistently.

CLAIM: Credential-file theft can retain authority after conversion, so the stated 24-hour blast-radius rationale does not bound continuing account access.  
EVIDENCE: S:31–33 acknowledges file theft; S:151–161 permits key-authenticated claim-start and explicitly treats TTY as bypassable; S:189 preserves the same key. W/middleware/auth.ts:174–192 continues accepting an active key after the account changes plan, and W/routes/account-items.ts:19–20 scopes history to that account. A thief can start a replacement claim, obtain its code, verify their own unused mailbox, authorize, and poll; the original CLI continues using that same account.  
CONFIDENCE: 0.94  
FALSIFIER: Conversion requires an independent possession factor unavailable from the stolen file, or invalidates the stolen credential’s continuing authority.

CLAIM: The UTC counter design prevents an in-place reset race but omits creation of each day’s counter row.  
EVIDENCE: S:108–115 defines a day-keyed table and only `UPDATE ... WHERE day = ? AND captures < ?`, with zero changes mapped to 429. There is no daily insert/upsert in that section. Its day boundary also permits one budget immediately before midnight and another afterward; that follows the advertised calendar-day policy at S:110–111 and :136, rather than exceeding either day’s cap.  
CONFIDENCE: 0.97  
FALSIFIER: A specified atomic initialization/upsert admits the first allocation of a new UTC day while concurrent allocations remain capped.

CLAIM: The enforcement inventory leaves an authenticated Browser-rendering path outside the trial spend and viewport controls.  
EVIDENCE: S:95 and :245–249 name capture validation, while W/routes/router.ts:154–155 exposes `/api/v1/render`. W/routes/render.ts:33–35 accepts authenticated callers, :56–66 permits six viewports and full-page rendering, and :118–141 allocates storage and launches Browser without a quota check. W/middleware/auth.ts:174–192 accepts active API keys without excluding trial plans.  
CONFIDENCE: 0.96  
FALSIFIER: The spec explicitly rejects trial rendering or requires the shared trial budget, concurrency reservation, and viewport restrictions before this path allocates resources.

**VERDICT INPUT**

Q1: Core decisions survive, but added defaults and the unresolved/raw-key contradiction require correction (claim 1).  
Q2: Migration, cleanup, browser approval authentication, and pairing completion are insufficiently specified (claims 2–5).  
Q3: Locator-only conversion is barred by S:147–149; stolen-key persistence and uncovered rendering remain material risks, while UTC boundary bursts follow the stated policy (claims 6–8).  
Q4: The implementing agent must currently invent consequential authentication, transaction, expiry, and counter-initialization behavior (claims 2–5, 7).

**PUBLISH VERDICT: blocker** — Resolve these specification gaps before implementation proceeds without human review.
