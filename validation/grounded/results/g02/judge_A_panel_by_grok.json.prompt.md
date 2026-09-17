You are scoring one review memo against a sealed rubric. You are not a participant; do not add your own review.
Score ONLY from the memo text. Quote the memo span that earns each point; if no span supports a point, the score is 0.

=== SEALED RUBRIC ===
# G02 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (discriminator candidate: one verified live bug plus a bearer-link trap)
- **correct_call:** Viable **only with layered controls**: a 1-viewport trial
  quota; a fail-closed per-IP issuance cap; a global daily trial budget as a
  hard ceiling; an env kill switch that stops issuance only; the claim link
  must be a **non-bearer locator** (never a capability URL, never in tool
  text) with possession proof (the trial-key-authenticated CLI poll does the
  conversion; the browser click is consent only); same-key in-place
  conversion (email attached to the existing trial account, devices carried);
  rung 0 **complements** `whipnode init`, it does not replace it. A schema
  migration is needed because `accounts.email` is `NOT NULL UNIQUE`.
- **oracle:** built and launched 2026-08-20 in this shape (trial-key-
  authenticated device-code poll; commits 7177649 transactional claim-start
  and 6f9dda5 poll mint rollback); the capture-path limiter bug was fixed on
  main ("bugfix 3", `capture.ts:251-257` post-fix, `rate-limit.test.ts`).
  Artifact facts: see must_catch.
- **correct fix (the verified bug):** call `checkUploadRateLimit` on the
  capture path for trial and free plans and fail closed on a missing IP for
  trials; forward the client IP onto the MCP-internal Request.
- **must_catch:**
  1. **Real existing bug (all three original seats confirmed it):** the
     authenticated capture path checks only the account/day quota
     (`apps/worker/src/routes/capture.ts:233-236` calls `checkDailyUploadLimit`
     only); the per-IP limiter `checkUploadRateLimit`
     (`apps/worker/src/middleware/rate-limit.ts:16-58`) is called on the
     upload path (`apps/worker/src/routes/items-create.ts:70-73`) but never
     on captures.
  2. The MCP capture path forges an internal Request without
     `CF-Connecting-IP` (`mcp.ts:1008-1013`; the upload variant says so
     explicitly at `mcp.ts:953`), so any IP-scoped check sees no IP, and the
     limiter **fails open** on a missing IP (`rate-limit.ts:20-21`
     `if (!ip) return { allowed: true }`).
  3. `PLAN_LIMITS[plan] ?? PLAN_LIMITS.free` (`rate-limit.ts:70`) means an
     unregistered `trial` plan silently gets 50/day; `AccountPlan`
     (`packages/protocol/src/types.ts:32`) has no `trial`.
  4. `accounts.email` is `TEXT NOT NULL UNIQUE`
     (`apps/worker/migrations/0001_initial_schema.sql:9`): a no-email trial
     account needs a migration (nullable email; SQLite UNIQUE permits
     multiple NULLs).
  5. One capture can cost up to six sequential Puppeteer viewports
     (`capture.ts:546-548`, `:565-570`) with a 25 s wait per navigation
     (`capture.ts:25`): the trial must be 1 viewport and browser concurrency
     is the real cost centre, not per-hour list price.
  6. The claim link must not be a bearer: conversion happens on the
     trial-key-authenticated poll, not on the browser Authorize; a leaked
     locator or a phished click alone converts nothing.
  7. Kill switch = stop new issuance only; existing trial keys hit the wall
     copy; claimed accounts untouched.
- **landmine (confident wrong answers):** "the existing IP rate limiter
  already covers captures" (it does not run on that path); "put the claim URL
  in the MCP error text, it is single-use so it is safe" (single-use bearer
  in a transcript is still a bearer); "reissue a new key on claim" (breaks the
  agent's stored config; in-place conversion was the agreed shape);
  "rung 0 replaces init" (both shipped).
- **unverifiable:** the exact trial quota (3 vs 10 vs 25/day) was an owner
  call and telemetry never discriminated; do not score it.
- **outcome_source:** `2026-08-19-whipnode-trial-ladder/retro.md` (T0
  `verified: 1` capture.ts limiter bug; agreed_r0 list; T1 outcome: built
  as the trial-key-authenticated poll, limiter bug fixed on main;
  dissent_confirmed_for: possession mechanism); `grok-r0.md:13,18,68,74`;
  `codex-r0.md:4,9`.


=== MEMO (author unknown) ===
# Verdict - g02: a zero-click trial ladder for WhipNode (rung 0 + in-surface claim link)

<!-- A_panel arm. Filled mechanically from ledger.md rows by the orchestrator (adjudicate hat only). No new arguments. -->

## 1. Independent agreement (agreed-r0)

The panel concludes (both seats, Round 0, before exposure):
- A1/B1: one capture is unbounded in cost today - up to 6 viewports x 25 s (+3 s), fullPage allowed, OCR per png, browser closed only on the success path.
- A2/A3/B2 (IP clause): no per-IP control reaches the capture path; MCP-originated internal Requests carry no client IP, and the IP limiter allows when the header is absent.
- A4/B3 (evidence clause): only two of six documented ABUSE_BUDGET limits are enforced anywhere.
- A5 (constraint)/B4 (email clause): `accounts.email` is NOT NULL UNIQUE and no later migration changes it.
- A7 (reuse)/B7: rung 0 complements `whipnode init`; device enrollment and the atomic magic-token consume are reusable.

## 2. Resolved after Round 0

**Oracle-settled (`verified`, by inspection of the cited spans; oracles.md):**
- A6: an unknown `plan` falls back to `PLAN_LIMITS.free` (50/day) - rate-limit.ts:70, constants.ts:44.
- B2 (race clause): daily-limit check at capture.ts:234 precedes launch (:565) and insert (:663); MCP batches run under Promise.all (mcp.ts:319-320).
- B4: items rows store deterministic sha256 hashes of IP and User-Agent (capture.ts:653-680, hash.ts:20-25).
- B6: free tier is ttl 24 h / history false; cleanup deletes expired objects (constants.ts:43-51, cleanup.ts:7-8).
- A7 (enrollment clause): enrollment runs only for `auth.type === "api_key"` on initialize; session token is returned in the Mcp-Session-Id header (mcp.ts:335,386).

**Cross-examination-settled (`conceded`):**
- B2 -> conceded by A (R1_claude "CONCEDE": atomic reserve-before-launch, charge failures; spec amended).
- B4 -> conceded by A ("zero PII is false as stated"; pseudonymous, disclosed).
- B6 -> conceded by A (copy must say nothing expired is restored).
- A6 -> conceded by B ("no substantive disagreement").
- A5 sentinel feasibility -> conceded by B ("technically possible", prefers nullable); A8 "device possession alone does not defeat (b)" -> conceded by A.

## 3. Surviving dissent

- Q1 headline (B3 vs A verdict): A - survivable only after three fixes (atomic pre-launch reservation; trial capped at 1 viewport/no full-page/no 4k; browser close in `finally`) behind a global daily trial budget; B - capped pilot only, abuse profitability and literal zero PII unproven. Same fixes required by both. Cheapest discriminating test: run the capped pilot with the budget and measure actual browser-seconds and issuance against the budget.
- Q2 claim binding (A8 revised vs B5 revised): A - public locator + 15-min single-use token + CLI-displayed code, plan flips only on verified email, no loopback adapter (unreachable from devcontainers/SSH/hosted agents; new component); B - a transcript-visible code is a bearer credential when it shares the leaked surface, email proves mailbox control not installation ownership; require browser-session-bound proof from the victim's installation with relay resistance demonstrated (loopback is a candidate, undemonstrated). Cheapest discriminating test: a red-team exercise in which an attacker holds a trial key and generates a claim link + code and a victim follows A's flow; if the victim's email attaches to the attacker's account, A's control fails.
- Q3 migration semantics (A5 vs B7 revised): A - same account_id, same key, `mcp_devices` carried; B - preserve account/items, replace credentials, carry only the proven device with its key association updated (auth.ts:103-104 checks the parent key). Open sub-point A did not address: an existing email colliding with UNIQUE at claim time. Cheapest discriminating test: none mechanical - a design choice; the auth.ts:103-104 fact is settled.

## Recommendation

**Mode:** `decide-after-check` on Q2 (the red-team check above), `ship` with the agreed fixes on Q1 and Q3.

On Q1 the panel is unanimous that today's code cannot host rung 0 (A1/B1, A2/A3/B2, A6) and unanimous on the fixes (B2 conceded, A6, A1's viewport cap); the surviving split is framing (B3), settled by running the capped pilot. On Q2 both seats agree (A8 revised, B5 revised, A's concession) that transcript-visible material and device possession alone do not bind the claimant; they disagree on the remedy, which the named red-team check discriminates. On Q3 "complement" is agreed (A7/B7); the credential-carry choice (A5 vs B7) is the owner's, with the UNIQUE-email collision (B's unaddressed point) to be handled either way.

## Record

- Open (unexamined, not endorsed): B's attack on A's IP/UA-keyed reuse and issuance-budget-as-ceiling (raised in R1, A did not reply within the round).
- Verified vs merely agreed: verified = A1, A2, A3, A4, A6, B1, B2, B4, B6, A5 constraint, A7 enrollment clause, B7 pointers; agreed-only = "complement", the fixes list; contested = A8/B5, B3, A5/B7 semantics.
- Rounds run / stop rule hit: R0 (reused solos) + R1; stopped: no load-bearing flip, no new claim IDs.
- Transformations applied to relayed text: neutral labels, order shuffle, ORCH-SUMMARY (see transformations.log).

VERDICT INPUT: Q1 - not survivable on today's code; ship rung 0 only after atomic pre-launch reservation, a 1-viewport/no-full-page/no-4k trial plan with PLAN_LIMITS.trial defined, browser close on the failure path, and a global daily trial budget with an independent kill switch; whether that is "survivable" or "pilot only" is decided by running the capped pilot. Q2 - decide-after-check: link alone must be inert and plan must flip only after verified email (agreed); whether a CLI-displayed code suffices or installation-bound proof is required is settled by the named red-team check. Q3 - complement `init`; reuse enrollment and magic-token consume; preserve account and items; credential carry (same key vs reissue) and the existing-email collision are the owner's design calls.


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}