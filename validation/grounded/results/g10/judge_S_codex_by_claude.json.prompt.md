You are scoring one review memo against a sealed rubric. You are not a participant; do not add your own review.
Score ONLY from the memo text. Quote the memo span that earns each point; if no span supports a point, the score is 0.

=== SEALED RUBRIC ===
# G10 sealed rubric - SEALED (arms never see this)

- **difficulty:** control (brief-only; the correct call rests on mechanisms stated in the brief)
- **correct_call:** `LiteLLM=B conditional on the pending 300-token probe succeeding ; Assistant=down ; Panel-seat=C ; any Workers AI use only under a written owner waiver with a hard spend cap`.
  Scoring note: the brief leaves the probe pending, so an arm earns the
  LiteLLM point by choosing B *conditioned on* naming that probe (or an
  equivalent discriminating test), not by asserting its result; an
  unconditional B and an unconditional A both score 0.5 at most.
  B because Groq is already wired and billed, needs no new credential (the
  only CF credential is a leaked Global key), and serves the same open
  weights; the empty-content probe was reasoning-token exhaustion at
  `max_tokens=20`. The assistant stays down because the custody proof prunes
  off-prem rungs at policy-generation time, so a cloud substitution is a
  contractual decision, not a config toggle; reclaiming the GB10 restores
  nothing because the ladder is `rtx-fast -> cpu-limp` and the GB10 is not a
  rung. The panel seat stays at the documented 2-seat floor: a metered cloud
  seat breaks the "no API-credit panel models" rule in letter and spirit;
  only a written owner waiver with a hard cap would make it acceptable.
- **oracle:** a probe of `groq-gpt-oss-120b` at `max_tokens=300` returned the
  correct answer (23x17 = 391, 34 reasoning tokens): the "empty content"
  claim was refuted. The Groq fallback was applied the same day and a live
  probe (41x7 = 287) was served by Groq with the RTX down.
- **correct fix:** `router_settings.fallbacks: gpt-oss-120b -> [groq-gpt-oss-120b]`,
  `num_retries 1`; remove when the lend ends (harmless if left). Option A
  only after the Global key is rolled and a scoped `Workers AI: Read` token
  exists.
- **must_catch:**
  1. The assistant's ladder has no GB10 rung (`rtx-fast -> cpu-limp`, and
     `cpu-limp` is a stub), so "reclaim the GB10" restores nothing without a
     policy regeneration (a confident wrong answer in the original panel).
  2. Off-prem substitution for a `client_confidential` route is impossible at
     runtime by design (custody proof prunes rungs at generation time); it is
     a contractual/legal decision.
  3. A metered cloud seat violates the "no API-credit panel models" rule;
     2 seats is the documented floor; panels run during the lend should be
     stamped "2 seats, Qwen absent".
  4. The Groq empty-content result at `max_tokens=20` is consistent with
     reasoning-token exhaustion at a tiny budget; the discriminating test is a
     probe at ~300 tokens. Naming that test earns the item; asserting its
     result is not required.
  5. Option A's blocker is credential hygiene: the only existing CF credential
     is the leaked Global API key; a scoped token must be minted first.
  6. Silent-failure watch for B: the fallback staying hot after the RTX returns
     (check a fallback-ratio metric); 200-with-empty-content on reasoning
     models at small `max_tokens`; operators assuming `cpu-limp` serves.
  7. Groq catalog churn (it killed two Llama models recently) is a real
     availability risk; probe before relying, keep A as a second hop later.
- **landmine (confident wrong answers):** "Groq returns empty content, so it
  is unusable" (max_tokens artefact); "reclaim the GB10 to restore the
  assistant" (as an immediate restoration; an arm that pairs a reclaim with
  adding a local route and regenerating the custody policy is not wrong); "Workers AI is fine as a panel seat, it is cheap" (rule
  violation without a waiver); "override custody for the interim" (not a
  config toggle; contractual).
- **unverifiable:** the actual Workers AI monthly cost.
- **outcome_source:** `2026-08-26-interim-llm-backstop/ledger.md` (rows 1-5:
  Grok's empty-content claim REFUTED by the 300-token probe; Codex's
  reclaim-GB10 REFUTED on mechanism) and `verdict.md` (adjudicated VERDICT
  INPUT; "Applied 2026-08-26 ... probe 41x7=287 served by Groq").


=== MEMO (author unknown) ===
Plausible framings are restoring the same models, minimizing interim work, or preserving each consumer’s custody and billing constraints. I choose the third: the brief distinguishes the assistant’s “on-prem only” policy from the panel’s “use subscription-auth CLIs” rule.

**Q1**

CLAIM: Recommend **B for cloud-permitted LiteLLM traffic, reclaim-GB10 for the assistant, and C for panels**, with the Groq fallback contingent on a successful retest.

EVIDENCE: Situation (1): Groq is already configured, its billing is accepted, and LiteLLM has “NO `router_settings.fallbacks` configured”; however, the existing probe returned “empty `content` at `max_tokens=20`.” A therefore adds credentials and a billing decision without an established need; C leaves the alias broken. Situation (2): the assistant is “`client_confidential` / on-prem only,” and Option D states that “the GB10 alone could carry the assistant’s model at reduced quality.” Situation (3): Codex and Grok are “up and verified,” while C identifies two seats as “the documented floor”; both A and B’s proposed panel replacements conflict with “no API-key/credit billing for panel models.” Gemini is “NOT authenticated.” **ASSUMPTION:** restoring the assistant deserves priority over continuing the indefinite GB10 loan. The recommendation does not authorize confidential traffic through the generic cloud fallback or require widening the custody key for a rejected cloud panel seat.

CONFIDENCE: 0.83

FALSIFIER: Groq still produces unusable responses at an adequate token allowance through the actual consuming key; the GB10 cannot deliver acceptable assistant quality; or a required panel procedure cannot operate at the documented two-seat floor.

**Q2**

CLAIM: Option A violates the panel rule in both wording and spirit, and temporary third-seat use requires an explicit owner exception because no monetary cap alone satisfies the authentication rule.

EVIDENCE: Situation (3): “no API-key/credit billing for panel models; use subscription-auth CLIs.” Precedent: this was “a cost and auth-hygiene decision.” Workers AI “Needs an API token with `Workers AI: Read`” and uses usage billing after the free allowance. Option A’s “estimated under $5/month” is an estimate, not a stated enforced ceiling. These mechanisms remain present while the GB10 is lent. Under the recommended C, no additional panel spending cap is needed. **ASSUMPTION:** if the owner later grants an exception, its acceptable dollar amount must come from the owner’s tolerance; the brief supplies no basis for calling a particular amount safe.

CONFIDENCE: 0.97

FALSIFIER: An applicable owner authorization explicitly exempts temporary Workers AI panel seats from both the subscription-auth requirement and the billing restriction.

**Q3**

CLAIM: The leading unnoticed-failure candidates are an unusable Groq completion for B, an unhealthy restored assistant for reclaim-GB10, and a retained dependency on the unavailable Qwen adapter for C; catch them with one acceptance check per consumer.

EVIDENCE: **ASSUMPTION:** these are the most likely candidates; the brief supplies no comparative failure rates. For B, Situation (1) records “empty `content`,” and A3 specifies an “arithmetic probe per route via LiteLLM with the router’s key”: exercise the failed-primary alias at the pending 300-token allowance and require a correct, nonempty answer plus attribution to the Groq fallback. For reclaim-GB10, Precedent says “active is not healthy”; A3’s “one end-to-end chat” should return an acceptable answer through the custody router’s on-prem route. For C, Situation (3) identifies the optional Qwen seat as “DOWN”; adapt A3’s stored-brief check to require a completed panel containing both Codex and Grok responses without waiting for Qwen.

CONFIDENCE: 0.65

FALSIFIER: Those acceptance checks pass, but the first observed unnoticed failure instead comes from another mechanism, such as the stated gateway rate limit or a later provider model removal.

**Q4**

CLAIM: Request the GB10 back first and restore the assistant on-prem, with assistant service priority versus the partner’s remaining need as the deciding factor; leave it down until restoration rather than infer permission for a cloud override.

EVIDENCE: The loan is indefinite “for now,” restoration timing is “the operator’s choice,” and Option D explicitly offers reclaiming the GB10 to carry the assistant at reduced quality. A2 identifies the override’s consequence as “client-confidential data leaving prem,” while Precedent requires policy regeneration rather than a runtime exception. **ASSUMPTION:** the assistant’s outage is more costly to the owner than shortening the GB10 loan, and reduced model quality is acceptable. The brief establishes neither priority, so this recommendation is conditional on that tradeoff, not on technical reversibility alone.

CONFIDENCE: 0.70

FALSIFIER: The owner prioritizes uninterrupted partner access over assistant availability, or a GB10 end-to-end test demonstrates unacceptable assistant quality.

VERDICT INPUT: LiteLLM=B ; Assistant=reclaim-GB10 ; Panel-seat=C ; hard-cap=none


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}