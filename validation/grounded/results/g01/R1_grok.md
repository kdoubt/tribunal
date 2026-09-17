I'll inspect the cited spans so the cross-exam is grounded in the artifact, not the other seats' wording.I have the cited spans; next I’ll check key-creation, grouping, and whether capture launch is inside the viewport loop.1. **ATTACK**

**Peer-1.** The negative fact is true: `mcp.ts:673-674` prints plan + today's count, not remaining/`PLAN_LIMITS`. That does **not** make status the upsell home — refusal copy already lives at `rate-limit.ts:85`. Remaining-on-status is a diagnostic, not an upgrade surface. 0.86 is fine for the gap, not for “already has a home.”

The attached FLOW is the real miss. Steps 4 and 6 are **two** human acts (Approve, then email click). Account insert is `auth-verify.ts:43-51`, so the named “one consent” click is not the creating act, and it runs before the inbox is proven. Step 5 is not “verbatim plus one column”: `auth-magic-link.ts:11` parses `{email?}` only. Step 8 `handleCreateApiKey` “no change” **403s** returning free users (`api-keys.ts:42-46`). Step 11 dropping the raw key after `initialize` strands `auth.ts:36-45` (expired device falls through to `X-API-Key`). Silent quota skip at `mcp.ts:369-375` is inherited. PKCE `code_verifier` is the wrong primitive (RFC 8628 already has a server-issued `device_code`). Free **15 captures/day** (~450/mo) and **2 viewports** fight their own cost thesis. Upsell #4 is false: grouping is per-device (`mcp.ts:700-701`), not per-key.

**Peer-2.** Right, and 0.9 is earned (`constants.ts:48`, `mcp.ts:352`, `api-keys.ts:42`). The FLOW still pretends mint-via-`handleCreateApiKey` is always legal. That is a spec hole, not a claim error.

**Peer-3.** Thesis holds; the pointer does not. `puppeteer.launch` is **outside** the loop (`capture.ts:565`; loop at `:567`). Cost still scales (`newPage` + `R2.put` per viewport; `QUEUE.send` at `:706`). 0.85 is a hair high for a mis-cited loop.

**Peer-4.** Right on the magic-link path (`auth-magic-link.ts:26` vs unread `:47`). 0.9 is fair. Overbreadth: post-account IP caps already exist (`ABUSE_BUDGET` 20 uploads/IP/h). The hole is issuance, not the whole surface.

**Peer-5.** Numbers mismatch (`pricing.tsx:47` 100 MB vs `constants.ts:55` 250 MiB). “Dishonest-by-staleness” overclaims: Pro is a disabled TBD card (`pricing.tsx:43-46`) under an early-access banner. Fix copy; don’t indict the funnel. 0.87 is too high for the moral frame.

**Peer-6.** New D1 table + no KV (`env.ts:3-7`) is right. “Only genuinely new **server** component” is not: start/poll routes, `/init` page, Turnstile, and the verify redirect they write in FLOW step 7. 0.88 is high for “table only.”

2. **CONCEDE**

- Status omits remaining/limit (Peer-1 fact).
- Free = 1 key × ≤10 devices (Peer-2).
- Captures need their own meter (Peer-3).
- Magic-link is per-email; `ip_hash` unread (Peer-4).
- Pro item copy is stale vs 250 MiB (Peer-5); code wins, `pricing.tsx` must be fixed before recipe copy ships.
- Pairing state is a new D1 table; poll is a new endpoint (Peer-6).
- Poll must be bound by a server-issued secret, not the public user-code.
- `credentials.json` mode 0600 beats shell rc.
- Refusal text should name remaining vs limit.

3. **REVISE**

- **Own-3** (0.78→0.82): pairing table + `/init`; poll auth is a **server-issued `device_code`**, not PKCE. Reuse magic-link, one key, `mcp_devices` session.
- **Own-4** (0.82→0.90): text unchanged; Peer-2’s pointers close it.
- **Own-5** (0.80→0.78): Pro item cap in the table is **250 MiB** (code). Add a **daily** capture sub-quota (≤10/day) under ~50/month; keep **1 viewport** as the free wall. Reject 15/day and 2 free viewports.
- **Own-6** (0.86→0.84): wall copy remains the upsell; `whipnode_status` may add remaining/limit as a diagnostic, not a nag.
- **Own-1, Own-2, Own-7, Own-8:** no revision. Own-2 stands: Authorize **after** `wn_session`/verify. Email click is identity (`email.ts` “Sign in”), not device consent.

4. **VERDICT INPUT**

- **Q1:** `npx whipnode init` / unauth MCP → device-code + `/init`; magic-link then one **Authorize**; reuse 1 write key; poll returns session (raw key once if new); store `credentials.json` 0600 (keep key for 90-day re-enroll).
- **Q2:** Free: 50 uploads/day, ≤10 captures/day (~50/mo), 1 viewport, 25 MB, 24 h, OCR, 1×3 devices; walls at extra viewport/4k, TTL, bytes, 4th device; recipe on whipnode.com; fix `pricing.tsx` to 250 MiB.
- **Q3:** Turnstile on `/init`, 10 min code, 3 free devices, hashed IP; agent starts/polls/captures only; human clicks Authorize and the email link; existing revoke/90-day sliding expiry.
