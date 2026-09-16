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
