# Verdict - g01 `whipnode init` onboarding flow

Built exclusively from ledger rows (results/g01/ledger.md). Panel seats: two vendors (A, B). No new arguments.

## 1. Independent agreement (agreed-r0)

The panel concludes (both seats, before cross-exposure): device enrollment on MCP `initialize` already mints a 90-day session and a device row, so init needs a way to obtain the first key, not a new device-auth system (A1, B3); the new server piece is a short-lived pairing/init-code table in D1 plus a browser consent page, reusing magic-link, the single free API key and `mcp_devices` (A2, B3); the signup surface today is per-email rate limiting only, with `ip_hash` stored but unread and no Turnstile/captcha anywhere (A3, A4, B7-facts); free segregation is one key with per-device rows (A5, B4); captures are the expensive unit yet are gated only by the shared 50 uploads/day counter, so a capture meter is needed (A6, B5-mechanism); `whipnode_status` and the refusal string are the existing no-dashboard surfaces and neither shows the remaining budget (A8, B6-fact); no anonymous trial key and no unattended account creation (A framing, B2).

## 2. Resolved after Round 0

**Oracle-settled (`verified`):** every cited pointer (13 + 29) opens; A1-A8 and B1-B8 factual content confirmed by inspection (oracles.md). A6's pointer is imprecise (browser launch at capture.ts:565 is once per capture, the viewport loop starts at :567) but the cost thesis stands. A's attack on B8 (revoked key may leave a live device session) is refuted: auth.ts:79-99 joins `api_keys` and rejects `key_revoked`. B's four attacks on A's FLOW SPEC are confirmed: magic-link parses `{email}` only (auth-magic-link.ts:11); a returning free user's key creation 403s at the cap (api-keys.ts:42-46); grouping is per device (mcp.ts:700-701); expired devices fall through to X-API-Key, so the raw key must be kept (auth.ts:36-45). A's attack on B4 is confirmed: `Mcp-Session-Id` is minted by the server on `initialize` (mcp.ts:386) and cannot be pre-written into a static header. A7 is confirmed and widened by A (pricing.tsx:47 100 MB vs constants.ts:55 250 MiB; terms.tsx:61 "No API access" vs api_access true).

**Cross-examination-settled (`conceded` / `revised`):** B conceded A1-A6 and A8's fact (R1_grok CONCEDE); A conceded B1, B2's no-anonymous half, B8's column set, B5's terms.tsx staleness, B7's agent-may-poll rule (R1_claude CONCEDE). B revised B3 (0.78→0.82, server-issued device_code not PKCE), B4 (0.82→0.90), B5 (0.80→0.78, Pro item 250 MiB, daily capture sub-quota added), B6 (0.86→0.84, status may show remaining/limit as a diagnostic). A revised A7 (0.87→0.95, widened) and its FLOW step 9 (poll returns raw key once). Converged: a DAILY capture sub-quota; status shows remaining vs limit and refusal names the limit; poll must be bound by a server-issued secret; credentials file 0600; fix pricing/terms copy before the recipe ships.

## 3. Surviving dissent

- D1 consent placement — A: Approve click (Turnstile) on the init page before the magic-link email, account created on link click; B: Authorize click after session/verify, email click is identity only. Cheapest discriminating test: count human acts in each flow against the brief's "exactly one clear moment" and the USER-FACT that account creation must follow a human act; a paper walk-through of both flows against auth-verify.ts:43-50 settles which click is the creating act.
- D2 poll payload — A: raw key once, session negotiated on first initialize; B: device session from poll, raw key only if newly created. Test: implement the poll against mcp.ts:335-387 and check whether a pre-minted session can be enrolled without an initialize round-trip.
- D3 free device cap — A keep 10 (farmers buy accounts, not devices); B 3. Test: the first month's device-count distribution on free accounts.
- D4 capture quota and viewports — A 15/day + 2 viewports; B ≤10/day (~50/mo) + 1 viewport. Test: Browser Rendering cost per capture vs per extra viewport (capture.ts:565 once per capture; per-viewport cost is newPage + R2 put).
- D5 label of the status surface (diagnostic vs upsell) — substance converged; no test needed.
- A7 framing ("dishonest-by-staleness" vs "stale copy on a disabled TBD card") — fix agreed; framing is taste.
- D6 issuance-time IP counter — A: read stored ip_hash; B: existing post-account IP caps suffice plus hashed IP; B concedes the issuance hole. Test: attempt N magic-link issuances from one IP with distinct emails.

## Recommendation

**Mode:** `ship` on the mechanism (bucket 1: A1/A2/B3, A5/B4, A3/A4/B7, B2), `decide-after-check` on the four parameters in bucket 3 (D1-D4), `ship` on the copy fixes (A7, B5-terms). The brief's tie-break criteria (under-two-minute, one consent moment, abuse cost > value, honest upsell, reuse existing tables) favour the shared skeleton and do not by themselves pick between the D1-D4 parameter values; those go to the owner, each with the test above.

## Record

- Open (unexamined, not endorsed): none.
- Verified vs merely agreed: verified = all 16 factual rows plus the R1 oracle findings listed in bucket 2; agreed-only = the design parameters inside D1-D6.
- Rounds run / stop rule hit: R0+R1; stopped: no VERDICT INPUT changed direction in R1, no new claim IDs.
- Transformations applied to relayed text: neutral Peer labels, shuffled order, ORCH-SUMMARY note; nothing else (transformations.log).

VERDICT INPUT: Q1 - device-code pairing over magic-link with one human Authorize click, one new D1 pairing table plus /init page, reuse the single free key and mcp_devices enrollment on initialize, keep the raw key in a 0600 credentials file for re-enrollment; consent placement (before vs after email verify) and poll payload are owner calls (D1, D2). Q2 - free stays 50 uploads/day, 25 MiB, 24 h, OCR, 1 key; add a DAILY capture sub-quota (10-15/day, D4) and a viewport wall (1 or 2 free, D4); Pro = more viewports/4k, 50 MiB, 7 d, history, 10 keys; upsell = refusal text naming remaining vs limit plus whipnode_status showing the budget; recipe on whipnode.com; fix pricing.tsx:47 (250 MiB) and terms.tsx:61 before launch. Q3 - Turnstile on the consent page, 10-minute pairing TTL, an issuance-time IP check (D6), agent may start/poll pairing and capture but never reads mail or clicks the link; store only email, key hash, device/ip/ua hashes and 24 h items; existing revoke, key cascade and 90-day sliding expiry suffice; free device cap 3 vs 10 is an owner call (D3).
