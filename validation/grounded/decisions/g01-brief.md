# Decision G01 - `whipnode init`: the CLI/MCP first-run onboarding flow

## Artifact(s)

Paths are relative to `./artifact/` (a WhipNode monorepo checkout: `apps/web` =
TanStack site, `apps/worker` = Cloudflare Worker API + D1). Read anything in
the tree; nothing outside it is in bounds. Study at minimum:
`apps/worker/src/routes/auth-magic-link.ts`, `auth-verify.ts`,
`mcp-devices.ts`, `mcp.ts`, `capture.ts`, `apps/worker/src/middleware/auth.ts`,
`apps/worker/migrations/` (auth, api-key, device tables),
`packages/protocol/src/constants.ts`, `apps/web/app/routes/pricing.tsx`, and
`docs/` if useful.

Facts supplied by the owner (USER-FACT):
- WhipNode is a "context relay for AI agents": capture/render/upload/fetch via
  MCP tools plus a web dashboard, OCR prompt packs, multi-viewport captures.
  Existing infrastructure: accounts, magic-link email auth, an `api_keys`
  table, an `mcp_devices` table (pairing, ip_hash, expires_at, revocation), a
  pricing page with a $0 free tier and a "pro" tier whose price is TBD, and a
  50 MB/file limit already gated behind paid plans.
- Goal: design the `whipnode init` onboarding flow - a CLI/MCP first-run flow
  that takes an agentic-CLI user from nothing to a working, properly
  segregated free-account API key in under about two minutes, with explicit
  human consent. Motives: adoption (agent users, and adopters of an external
  review methodology arriving via a whipnode.com recipe page) and an upsell
  lane to pro.
- Standing constraints: unattended account creation without human consent is
  unacceptable; the public signup surface must resist bot and free-tier
  farming; the external methodology's public repo will never name WhipNode,
  so the funnel entrance is whipnode.com content plus the MCP/CLI itself.
- Operator practice for that methodology: the orchestrator captures, seats
  receive static evidence (an injection-egress-safe pattern).

## Question under review

- **Q1 The flow.** Specify `whipnode init` end to end: trigger points (CLI
  command, MCP first use with no key, dashboard copy-paste), auth mechanism
  (reuse magic-link, add device-code pairing, anonymous trial key with
  claim-later upgrade), the consent moment (where exactly the human says yes
  and to what), key issuance and storage (file, env, per-device row), naming
  and segregation (per-device keys per the existing `mcp_devices` model?),
  and failure/retry UX. Ground every choice in what the code already supports
  versus what must be built.
- **Q2 Tier and upsell lane.** Shape the free tier for this funnel
  (captures/month, retention, viewports, OCR, rate limits) and name the
  natural upgrade triggers where an agent user honestly hits a wall, with no
  dark patterns. Where does the upsell surface live for a user who never opens
  the dashboard, and what does the whipnode.com recipe page contain? Judge
  against: adoption first, revenue second, no dark patterns.
- **Q3 Abuse and consent guardrails.** Bot/farming resistance for the init
  path (is email verification enough; device caps per account; IP heuristics;
  Turnstile; the cost asymmetry of captures); what the agent may do
  autonomously versus what must be human-in-the-loop; data minimization (what
  a free init account stores); revocation and expiry defaults.

## Decision criteria (owner-supplied)

Under-two-minute zero-to-first-capture; explicit human consent at exactly one
clear moment; abuse cost exceeds abuse value on the free tier; upsell honest
and visible without nagging; small-team maintenance on the existing Cloudflare
Worker + D1 stack; reuse existing tables and flows over new systems.

## Constraints

- No unattended account creation. The one consent moment must be a human act.
- The methodology repo stays uncoupled; the recipe lives on whipnode.com.

## Output contract

Maximum 8 claims, each as:

```
CLAIM: <one sentence>
EVIDENCE: <file:line or verbatim span in ./artifact/, or USER-FACT, or ASSUMPTION, or SPECULATIVE (name the unreadable dependency), or EXTERNAL with source>
CONFIDENCE: <0-1 probability, calibrated>
FALSIFIER: <what concrete observation would prove this claim wrong>
```

Then a FLOW SPEC (numbered steps, at most 12, from "user has nothing" to
"first capture returned"), a TIER TABLE (free vs pro, 5-8 rows), UPSELL
TRIGGERS (at most 5, one line each), and **VERDICT INPUT**: one line per
question. Maximum 1200 words.
