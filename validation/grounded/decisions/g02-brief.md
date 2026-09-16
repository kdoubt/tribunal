# Decision G02 - a zero-click trial ladder for WhipNode (rung 0 + in-surface claim link)

## Artifact(s)

Paths are relative to `./artifact/` (WhipNode monorepo: Cloudflare Worker +
D1 + Puppeteer captures). Read anything in the tree; nothing outside it is in
bounds. Key files: `apps/worker/src/routes/capture.ts`, `mcp.ts`,
`auth-magic-link.ts`, `auth-verify.ts`, `apps/worker/src/middleware/rate-limit.ts`,
`middleware/auth.ts`, `packages/protocol/src/constants.ts` (PLAN_LIMITS and
anonymous IP budgets), `apps/worker/migrations/`, `docs/ABUSE_PREVENTION.md`.

Context supplied by the owner (USER-FACT):
- A prior design for `whipnode init` was adopted: device-code pairing wrapping
  the magic link, one consent click ("Authorize this device"), one free key
  plus an `mcp_devices` enrollment per installation.
- That design froze "explicit human consent for account creation" as a
  constraint, derived from scenarios involving the adopter's email/PII. New
  framing: a trial account containing zero personal data (no email, bound only
  to an installation-generated credential) has no one whose consent is
  missing; it is a rate-limit token, not a signup. Anonymous trial keys were
  rejected earlier mainly on abuse grounds, so abuse is today's centre
  question; the consent leg is dissolved unless you can re-establish it.

Proposal under review (owner's): a three-rung ladder replacing or augmenting
`whipnode init`:
- RUNG 0 (zero-click): first MCP tool use with no key auto-provisions an
  anonymous trial credential (about 10 captures/day, 24 h retention, no PII).
  The agent works within seconds of install.
- RUNG 1 (one-click claim): when the trial hits any wall (daily cap, expired
  item, device), the MCP/CLI error response itself carries a CLAIM LINK,
  surfaced in the user's CLI/IDE by their agent, that converts the anonymous
  account in place (email attach + a single Authorize click, history and key
  intact) to the full free tier (50/day per `constants.ts`).
- RUNG 2: pro upgrade, unchanged.

## Question under review

- **Q1 Rung-0 abuse economics.** Is a zero-click trial tier survivable for a
  small product paying for Puppeteer compute? Ground in code: what does one
  capture cost in bounded resources (timeouts, viewport count, concurrency,
  existing anonymous IP budgets)? Specify the issuance gate (what stops
  scripted mass provisioning: per-IP caps, ASN/bot-score checks, proof of
  work, a global daily trial budget, what the credential is bound to), the
  right trial quota shape, and the kill switch (how the owner turns rung 0 off
  if farmed, without breaking claimed accounts).
- **Q2 The in-surface claim link.** Design the claim mechanics: how the link
  is bound to the anonymous account; TTL; what the page shows; where email and
  Authorize happen. Security centre: the link appears in CLI/IDE output and
  agent context. Address both hijack directions: (a) someone else who sees the
  link (logs, screenshots, shared terminals, agent transcripts) claiming the
  user's trial account; (b) an attacker inducing a victim to click an
  attacker-generated claim link, attaching the victim's email to an
  attacker-held key. What binding proof (device possession, recent-activity
  display, code confirmation in the CLI) defeats each? Give the exact
  error-response copy the agent relays.
- **Q3 Ladder versus init.** Does rung 0 replace `whipnode init` or complement
  it? What of the prior design's device-code/approve-page work is reused by
  the claim flow? Migration semantics trial -> claimed (same key or reissue;
  `mcp_devices` rows carried?). What changes on the whipnode.com recipe page?

## Decision criteria (owner-supplied)

Abuse cost exceeds abuse value at every rung; zero PII stored pre-claim; the
claim link must be safe to display in logged or shared surfaces (assume
transcripts leak); time-to-first-capture under 30 s on rung 0; upsell honest;
small-team ops (Cloudflare Worker + D1, no new services); reversible (rung 0
can be disabled independently).

## Constraints

- No new services. Existing tables and flows preferred.
- Anything that becomes a bearer credential when pasted into a transcript is
  unacceptable.

## Output contract

Maximum 8 claims, each as:

```
CLAIM: <one sentence>
EVIDENCE: <file:line or verbatim span in ./artifact/, or USER-FACT, or ASSUMPTION, or SPECULATIVE (name it), or EXTERNAL with source>
CONFIDENCE: <0-1 probability, calibrated>
FALSIFIER: <what concrete observation would prove this claim wrong>
```

Then a LADDER SPEC (at most 12 numbered steps: rung-0 issuance -> wall ->
claim -> converted), a GUARDRAIL TABLE (threat -> control, at most 8 rows),
and **VERDICT INPUT**: one line per question. Maximum 1200 words.
