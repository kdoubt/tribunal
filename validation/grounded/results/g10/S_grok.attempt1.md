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
