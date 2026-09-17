# Verdict - g04: does the landing page speak to the agent-CLI adopter, and what ships before the trial ladder?

Built from ledger rows only. Seats A and B; authorship in ledger.md. Rounds: R0 + R1 (R1 for seat B completed 2026-09-17 after a provider usage cap; packet unchanged).

## 1. Independent agreement (agreed-r0)

The panel concludes, from claims both seats made before cross-exposure:
- A1/B1: the page uses agent language but its conversion path is human-first; the biggest gap is that the agent is never the first actor (no agent-initiated capture-and-read as the primary action).
- A2/B1/B3: capture is a buried scenario and a late MCP card, not a pasteable recipe; a primary trial instruction is missing.
- A4/B2: "No account needed" (`index.tsx:72`) contradicts the URL-capture login wall (`UploadForm.tsx:182-184`, `:540`) and the "Create an account to get your API key" install line (`:183-184`).
- A8/B5: "End-to-end encrypted" is unsubstantiated against the server-side OCR path; qualify or remove.

## 2. Resolved after Round 0

**Oracle-settled (`verified`):**
- A1, A2, A3, A4, A5 facts, A6 web-route block, A7 facts, A8 facts, B2, B3 facts, B6 prompt behaviour, B7 facts → every cited `index.tsx` / `UploadForm.tsx` / `CookieConsent.tsx` / `__root.tsx` span opened and says what the claim says (oracles.md). B4's premise (the Claude Code limitation) and the terminal-path effect of the consent gate are not settleable from the artifact; B8 mobile readability is NO-SIGNAL.

**Cross-examination-settled (`conceded` / `overturned`):**
- A3 conceded by B (absence of Codex/Grok guidance; not incompatibility).
- A5 conceded by B as "the URL invitation omits its account requirement"; A's "sells as equivalent" framing narrowed.
- A6 and B6 converged: both seats now state a web-route hard block on decline via `ConsentGate` and leave the terminal path unsettled (A6 0.85→0.88, B6 0.94→0.98).
- B3 conceded by A (signup-free trial instruction + recipe link before the MCP-key install), with A's note that a Claude-only instruction still leaves Codex/Grok unaddressed.
- B4's prescription conceded by A (lead with capture-and-read, demote manual upload); the "alleged limitation" warrant remains contested.
- B8 facts conceded by A (NO-SIGNAL on mobile terminal readability; no clipping claim).
- Overturned: none.

## 3. Surviving dissent

- A7 vs B7 (hero "See how to scale"): A - drop the hero pricing CTA; it points a zero-click trial at plans. B - keep pricing discoverable, rename the CTA and state anonymous-trial vs account limits beside it. Cheapest discriminating test: ship one variant per arm of the trial launch and compare anonymous-first-capture rate; or, pre-launch, a five-user task test ("start using this with your agent") counting how many land on `/pricing` first.
- A7 vs B (does the current page *actively hurt* the ladder, or is it a launch-readiness gap): settled only by the same launch measurement; no artifact check can distinguish them.
- B4 premise ("You can't paste screenshots into Claude Code" is an alleged limitation): A - the pack shows the line asserted, not false; B - the line should not be the hero's dependency. Discriminating test: a documented, current Claude Code capability check for image paste; if true, the line is accurate and the dispute is copy taste.
- B8 as a launch blocker: A - not a blocker under a one-person ship; B - verify copy-and-run on mobile before launch. Discriminating test: one real mobile capture of the terminal block, readable or not.

## Recommendation

**Mode:** `ship` after the agreed changes; `decide-after-check` on the hero pricing CTA.

The agreed bucket (A1/B1, A2/B3, A4/B2, A8/B5) plus the conceded items (A3, A5, A6/B6, B3, B4 prescription, B8 facts) give a converged change list before the trial launch: an agent-first hero that leads with capture-and-read, a primary signup-free trial instruction with a recipe link ahead of the MCP-key install, explicit disclosure of the URL-capture and MCP account requirements next to the "No account needed" promise, a qualified or removed "End-to-end encrypted" chip, and a consent flow that no longer hard-walls the web routes on decline. The check that remains is the hero "See how to scale" CTA (A7/B7): drop it or relabel it, decided by the launch measurement named in bucket 3. B8's mobile verification and B4's premise are cheap checks that do not change the change list.

## Record

- Open (unexamined, not endorsed): none - every row was agreed, verified, conceded, or is in bucket 3.
- Verified vs merely agreed: verified = A1-A8 facts, B2, B3 facts, B6, B7 facts (oracles.md); agreed-only = the "biggest gap" framing (A1/A2/B1), B4's prescription, B5/A8's risk judgment.
- Rounds run / stop rule hit: R0+R1; stopped: no load-bearing claim flipped, no new claim IDs in R1. Seat B's R1 was a provider-limit death on 2026-09-16 (three empty attempts) and completed on attempt 4 on 2026-09-17 from the unchanged packet; seat A's R1 was not read until then.
- Transformations applied to relayed text: neutral Peer labels + shuffle; ORCH-SUMMARY ledger note (agreed-r0 list + oracle facts); nothing else (transformations.log).

VERDICT INPUT: Q1 - agent language, human-first conversion; the biggest gap is no agent-initiated, pasteable capture-and-read first action (Claude/Codex/Grok) above the fold. Q2 - before launch: agent-first hero leading with capture-and-read; a primary signup-free trial instruction with expected output and a recipe link before the MCP-key install; disclose the URL-capture and MCP account requirements beside "No account needed"; qualify or remove "End-to-end encrypted"; stop hard-walling web routes on cookie decline; the hero "See how to scale" CTA is decide-after-check (drop vs relabel with trial/account limits). Q3 - yes: the "No account needed" vs URL/MCP walls contradiction, the web cookie clickwrap with a hard block on decline, the unsubstantiated E2E chip, and "Create an account to get your API key" as the displayed prerequisite; whether the hero pricing CTA actively hurts the ladder is surviving dissent, and nothing on the page is shown to block the terminal path.
