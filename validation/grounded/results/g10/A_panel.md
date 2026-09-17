# Verdict - g10 interim LLM backstop while both estate GPUs are lent

Built by the orchestrator (adjudicate hat only) from ledger.md rows. No new arguments. Seats: two vendors, R0 + R1.

## 1. Independent agreement (agreed-r0)

The panel concludes, from claims both seats made before cross-exposure:
- Q2: Option A (Workers AI) as a panel third seat violates the owner's "no API-key/credit billing for panel models; use subscription-auth CLIs" rule in spirit as well as wording; a dollar cap does not cure the auth-hygiene half; therefore the panel seat is C (the 2-seat Codex + Grok floor) while the GB10 is lent (A2, B2, A1, B1).
- Q4 (override part): no cloud custody override for the assistant; a runtime "just this once" is impossible without regenerating policy (A4, B4).

## 2. Resolved after Round 0

**Oracle-settled (`verified`):**
- A2/B2 mechanism: brief:36-44 states the rule, the API-token requirement and the usage billing verbatim → the basis of the Q2 conclusion is verified by inspection. All 30 brief statements either seat relied on exist verbatim in the brief (oracles.md).

**Cross-examination-settled (`conceded`):**
- No Groq fallback (B) until the pending 300-token probe returns usable content through the consuming key: conceded by A (R1_codex CONCEDE, Own-1 revised to "B after verification"), held by B (R1_grok Own-1).
- LiteLLM=C rejected by both (it leaves a named production alias dead): B concedes in R1; A held it from R0.
- The deciding factor for reclaim-GB10 vs leaving the assistant down is the owner's stated priority of assistant uptime versus the partner's remaining need, not custody: B revised B4 to say so (0.80→0.78); A conceded "down" is defensible if partner access has priority and that the brief establishes neither priority.
- Q3 checks: an uncached arithmetic probe that asserts non-empty content and attributes the serving execution; one end-to-end assistant chat; a stored-brief panel run that verifies two completed seats and no local-seat call; recurring route checks for fallback drift (A revised Own-3 to this; B's B3 supplied the deployment-id and two-seat elements and holds).
- The $5/month figure is an estimate of volume, not an enforced cap (A concedes; B's cap proposal is a proposed ceiling, not a brief fact).

## 3. Surviving dissent

- **LiteLLM default while the probe is pending.** A (codex): B, after a verified probe, because Groq is already wired and billed (0.77). B (grok): A now, same weights via the gateway, B blocked until the probe passes and given Groq's catalog churn (0.68). Cheapest discriminating test: run the pending 300-token probe of `groq-gpt-oss-120b` through the router's actual key; a correct non-empty answer collapses the dispute to "B" (both seats' own stated conditions), a failure collapses it to "A after its own probe".
- **Assistant default pending the owner's instruction.** A: request the GB10 back (conditional on priority and acceptable quality, 0.70). B: leave down; reclaim only on an explicit owner instruction (0.78). Cheapest test: none technical; it is the owner's stated priority (both seats agree this is the deciding factor).
- **Hard cap.** A: none needed under its recommendation. B: $5 for the loan window on the LiteLLM alias only (outside the panel rule). Cheapest test: owner's tolerance; the brief supplies no basis for a number.
- **Q3 LiteLLM-path mechanism.** B: 200-with-empty content or fallback/cache staying hot after the RTX returns. A: unusable completion, and a cached answer cannot establish health. Cheapest test: the same uncached, attributed probe both now specify, run once after the RTX returns.

## Recommendation

**Mode:** `decide-after-check` on Q1 (LiteLLM) and `human-call` on Q4 (assistant); `ship` on Q2 (panel = C, no cap needed for the panel) and on Q3's check set.

Q2 and the no-override half of Q4 are bucket-1 conclusions (A2, B2, A4, B4). The LiteLLM option turns entirely on one check both seats named and neither has seen: the pending 300-token probe (A1, B1; conceded conditions in both R1s). The assistant's default is not decidable from the brief; both seats now say the deciding factor is the owner's priority statement (A4, B4 revised), so it is the owner's call. The Q3 check set is the union both seats accept after Round 1 (A3 revised, B3).

## Record

- Open (unexamined, not endorsed): none.
- Verified vs merely agreed: verified = A2/B2 basis, brief quotations (30/30); agreed-only = panel=C, no override, LiteLLM≠C.
- Rounds run / stop rule hit: R0+R1; stopped: no load-bearing claim flipped (both VERDICT INPUT lines unchanged), no new claim IDs.
- Transformations applied to relayed text: neutral Peer labels, order shuffle, ORCH-SUMMARY ledger note (settled items + oracle results); nothing else (transformations.log).

VERDICT INPUT: LiteLLM=decide-after-check (B if the pending 300-token Groq probe passes through the router's key, else A after its own probe; never C) ; Assistant=down-unless-owner-instructs-reclaim (no override; human-call on priority) ; Panel-seat=C ; hard-cap=none for the panel, $5-on-the-LiteLLM-alias is surviving dissent
