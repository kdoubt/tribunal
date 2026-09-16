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
