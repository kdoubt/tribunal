A_2seat = union of two isolated Round 0 memos, no ledger, no Round 1

=== Memo 1 ===

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

=== Memo 2 ===

**Framings considered:** (1) same-model continuity → A for every consumer; (2) zero-new-vendor → B for LiteLLM, C elsewhere; (3) policy-literal: cloud only where neither custody nor the panel-billing rule applies; (4) hardware-first → reclaim GB10; (5) availability-first, including a custody override. **Chosen: (3).** The brief’s constraints are per-consumer and contradict a single option. Unifying them is the error.

---

**Q1**

```
CLAIM: LiteLLM alias gpt-oss-120b → A (CF same-weights fallback); assistant → leave down (not A/B/D-override); panel third seat → C (2-seat floor); do not use B until the empty-content probe is actually passed.
EVIDENCE: LiteLLM has no `router_settings.fallbacks` and the local `gpt-oss-120b` path is a connection error; CF catalog includes `@cf/openai/gpt-oss-120b` (same model as the RTX) and fallback “triggers only on failure, so when the RTX returns it re-takes traffic automatically.” Groq already carries `groq-gpt-oss-120b` but “a probe … returned empty `content` at `max_tokens=20`” and “retest at 300 is pending”; Groq “recently killed two Llama models the config still references (dead aliases … silent failures)” and precedent is “probe before relying.” Assistant ladder is `rtx-fast -> cpu-limp` with `cpu-limp` a stub; route is `client_confidential` / on-prem only; “a custody proof prunes off-prem rungs at policy-generation time”; “a runtime ‘just this once’ is architecturally impossible without regenerating policy.” Panel: “Codex and Grok are up and verified”; “panels run 2-seat (Codex + Grok, the documented floor)”; standing rule “no API-key/credit billing for panel models; use subscription-auth CLIs”; Gemini CLI “installed but NOT authenticated.”
CONFIDENCE: 0.72
FALSIFIER: Groq retest at 300 returns non-empty, correct content and CF arithmetic probe fails or cannot be reached with a scoped Workers AI token; or the owner documents that the assistant must be up during the loan and refuses to reclaim GB10.
```

Why not the others, in short: **B for LiteLLM** is the least-new-surface path but is currently an unverified reasoning-model 200-with-empty-body, on a vendor that already left dead aliases in this estate. **C for LiteLLM** leaves a named production alias on connection error while a same-weights, fail-open-on-local-failure fallback exists and rolls back in `<2 min`. **A/B for the assistant** are custody-pruned unless policy is regenerated. **A/B for the panel** are API-credit seats (see Q2). **Gemini** matches the subscription-auth rule but is not authenticated and is not the missing Qwen-heterogeneous seat.

---

**Q2**

```
CLAIM: Yes — Option A as a panel third seat violates the standing rule in spirit (usage billing + API token, not a subscription CLI); it is not acceptable for that seat while GB10 is lent; a dollar cap does not repair the auth-hygiene half of the rule.
EVIDENCE: Owner rule: “no API-key/credit billing for panel models; use subscription-auth CLIs.” CF billing is “usage-based, 10,000 neurons/day free then $0.011/1k neurons.” Precedent: the rule “was a cost and auth-hygiene decision, not a data-custody one.” Documented floor is 2-seat Codex+Grok. LiteLLM already has `max_budget: $100/30d` (too loose for an interim CF fallback). Volume estimate in the brief: “under $5/month.”
CONFIDENCE: 0.86
FALSIFIER: Owner writes an explicit, time-boxed exception that panel seats may use Workers AI API tokens during GPU loans, or evidence that Workers AI is billed as a subscription rather than neurons/credits.
```

A **LiteLLM** fallback is outside that rule (the rule names panel models). If the owner still wants a CF spend ceiling on that alias, cap it at **$5** for the loan window (brief’s own volume estimate), not the existing $100/30d LiteLLM budget.

---

**Q3**

```
CLAIM: LiteLLM/A fails silently as HTTP 200 with empty or reasoning-only `content`, or as fallback/cache remaining hot after the RTX is back; the catch is an arithmetic probe that asserts non-empty numeric content and the serving deployment id. Assistant/down fails silently if `cpu-limp` returns a stub success rather than a hard error; the catch is one end-to-end chat expecting a hard failure. Panel/C fails silently if the adapter still calls :8200 and a 3-seat report is emitted on a 2-seat run; the catch is the stored-brief adapter run showing two seats and no :8200 call.
EVIDENCE: Groq probe of the same open-weights 120b class “returned empty `content` at `max_tokens=20` (reasoning model).” A2: “risk = silent cost/quality drift if nobody notices the fallback stayed hot.” LiteLLM has “Redis cache, analytics logging.” A4: AI Gateway “50 req/60 s limit and a 1 h cache.” Precedent: Groq Llama removals → “dead aliases … silent failures.” Assistant: `cpu-limp` “is a code stub”; A3 verification for the assistant is “one end-to-end chat.” Panel A3: “run the adapter against a stored brief.” `cpu-limp` success-shaped return is ASSUMPTION (stub body not in the brief).
CONFIDENCE: 0.64
FALSIFIER: CF/LiteLLM probe shows non-empty content and logs the local deployment whenever :8000 is up; cpu-limp always errors closed; panel adapter refuses to start if :8200 is down and labels the run 2-seat.
```

---

**Q4**

```
CLAIM: Leave the assistant down; do not regenerate policy for an off-prem rung; reclaim GB10 only if the owner separately decides the assistant must be up before the partner is finished — the deciding factor is custody (client-confidential data must not leave prem), not model quality or cost.
EVIDENCE: Route is “`client_confidential` / on-prem only by custody policy”; “policy forbids substituting any cloud model unless the owner overrides custody.” A2: custody override = “client-confidential data leaving prem (contractual/legal, not technical).” Runtime override “architecturally impossible without regenerating policy (by design).” Owner already handed both GPUs “interactively and for an indefinite ‘for now’”; “a `close` script restores prod at any time.” GB10 “alone could carry the assistant’s model at reduced quality” — that is the on-prem recovery path, not a default. No owner override is stated in the brief.
CONFIDENCE: 0.80
FALSIFIER: Owner issues a written temporary custody override, or instructs the partner to return the GB10 because assistant uptime during the loan is required.
```

---

VERDICT INPUT: LiteLLM=A ; Assistant=down ; Panel-seat=C ; hard-cap=$5
