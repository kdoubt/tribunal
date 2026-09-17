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
