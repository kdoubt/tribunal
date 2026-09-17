## 1. ATTACK

**Peer-1 (0.88)** — the data-minimization list is right; "revocation/expiry already exist" is the over-claim. `mcp-devices.ts:70-74` flips device status, and `mcp.ts:282-284` re-checks the device on each session lookup — fine. But "key cascade-revoke" (`api-keys.ts:139-143`) is asserted, not shown, to invalidate *live* device sessions; if session validation resolves the device row without joining `api_keys.status`, a revoked key leaves a 90-day session alive. Peer-1's own falsifier names this and the evidence doesn't close it. 0.88 → ~0.7 until the join is quoted.

**Peer-2 (0.82)** — the key half is correct and matches my Own-5; the ASSUMPTION is load-bearing and false in the direction it matters. `Mcp-Session-Id` is *minted by the server* in a response header on `initialize` (`mcp.ts:386`); a CLI cannot pre-write it into a static `claude mcp add --header` config, because it doesn't exist until after the first handshake. So step 6's "write MCP header (`Mcp-Session-Id` or `X-API-Key`)" is unimplementable as a disjunction: init writes the **key**, and the session is negotiated automatically. Drop the assumption; the claim survives.

**Peer-3 (0.80)** — "1 viewport free" is the wrong wall on the wrong axis. ORACLE confirms the browser launches **once per capture** (`capture.ts:565`, viewport loop at `:567`); additional viewports are a `setViewport` + screenshot, not a second Puppeteer launch. Gating the cheapest axis while leaving the expensive one (captures) metered only by the shared 50-uploads/day counter inverts the cost model and amputates the feature that sells Pro before the user has seen it work. Free should be 2 viewports. Also, `50 captures/month` invents a rollover window the codebase has no idiom for; `checkDailyUploadLimit` (`capture.ts:234`) is daily. Monthly = new column, new reset semantics, and ~1.6/day for a bursty first session — it fires the wall *during* the two-minute onboarding.

**Peer-4 (0.77)** — Turnstile: agreed (my Own-4). The 3-device cap is the weak leg and is doing no abuse work: a farmer buys **accounts**, not devices; the cap only taxes the legitimate laptop + desktop + CI + devcontainer user. Device rows are not the cost unit. Keep 10.

**Peer-5 (0.86)** — overstates "already." `rate-limit.ts:82-85` speaks only at refusal, and `mcp.ts:672-673` prints plan and today's count with **no limit rendered** (my Own-8). The surface exists; the *number* does not. As a description of today, 0.86 is too high; as a design target it's right.

**Peer-6 (0.95)** and **Peer-7 (0.90)** — no attack; evidence resolves.

## 2. CONCEDE

- Peer-6: nothing in-tree reaches capture unauthenticated; the CLI is Future (`ROADMAP.md:36`). Ground the whole flow on that.
- Peer-7: no anonymous trial key, no auto-insert into `accounts`; single human click.
- Peer-1: the minimized column set, and that `/init` must persist nothing beyond it.
- Peer-3: `terms.tsx:61` "No API access" is a second staleness bug alongside my pricing one.
- Peer-4: agent may start and poll pairing; must never read mail or click the link.

## 3. REVISE

**Own-7** → new text: "Public copy already misstates the product in two places — `pricing.tsx:47` (100 MB/item vs 250 MiB in `constants.ts:55`) and `terms.tsx:61` ('No API access' vs `PLAN_LIMITS.free.api_access`) — so funnel copy is dishonest-by-staleness before build." **0.87 → 0.95** (ORACLE-confirmed; scope widened).

**FLOW SPEC step 9** → poll returns the raw key **once** only; the session token is never pre-written (rebuts Peer-2's assumption). No other revision.

## 4. VERDICT INPUT

- **Q1:** Device-code pairing + magic-link, one Authorize click; new `init_codes` table only; poll returns raw key once; `initialize` handles device enrollment unchanged.
- **Q2:** Free = 50 uploads/day + **15 captures/day** (daily, not monthly), 2 viewports, 24 h TTL, 1 key × 10 devices; walls named with remaining-vs-limit in `whipnode_status`.
- **Q3:** Turnstile on `/init` + read the stored `ip_hash` + 10-min TTL; keep the 10-device cap; agent starts/polls only.
