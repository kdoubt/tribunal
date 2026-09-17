## Framing

Plausible framings: (i) **cost/effort-minimal** — pick whatever is already wired; (ii) **fidelity-preserving** — pick the identical model so behaviour doesn't drift; (iii) **interim-hygiene** — the horizon is "hours or days" (BRIEF.md:63), so minimize durable new artifacts (credentials, billing relationships, policy regenerations) that outlive the outage; (iv) **custody-first** — the binding constraint is the custody proof and everything else is downstream.

I adopt (iii) with (iv) as a hard side-constraint. Reason: fidelity does not discriminate here — both Groq and Workers AI serve the *same* open-weights `gpt-oss-120b` ("`groq-gpt-oss-120b` (same open-weights model…)", BRIEF.md:68; "`@cf/openai/gpt-oss-120b` … same model as the RTX", BRIEF.md:87–88) — so the real axis is what residue each option leaves when the `close` script runs. And A1/A2 mark every change reversible *except* client-confidential data leaving prem (BRIEF.md:129–130), which makes custody the only one-way door in the decision.

---

```
CLAIM: LiteLLM alias -> B (fallback to groq-gpt-oss-120b, contingent on the max_tokens=300 retest returning non-empty content); the assistant -> leave down and ask the partner for the GB10 back; the panel third seat -> C (run the documented 2-seat floor).
EVIDENCE: BRIEF.md:69-70 "Groq API key already present = Groq billing already accepted"; BRIEF.md:105-107 B "Requires widening one virtual key's model list"; vs BRIEF.md:103-104 A "Requires a new scoped CF token (mint, store), accepting usage billing". BRIEF.md:83 panel Qwen seat is "optional"; BRIEF.md:114 "panels run 2-seat (Codex + Grok, the documented floor)". BRIEF.md:118-119 "the GB10 alone could carry the assistant's model at reduced quality".
CONFIDENCE: 0.6
FALSIFIER: The retest at max_tokens=300 still returns empty `content`, or `groq-gpt-oss-120b` 404s like the removed Llama aliases (BRIEF.md:109-111) — either flips LiteLLM to A. Or the owner states the third seat is not optional for the panels actually scheduled during the interim — flips panel to gemini/A.
```

**Why not the others.** For the alias, A buys nothing B doesn't (same model), and costs a token mint against an account whose only existing credential is a *leaked* Global API Key pending roll (BRIEF.md:93–94) — do the roll on its own merits, don't couple it to an outage fix. C leaves a named alias dead while a same-model substitute sits pre-wired one config line away. For the panel, A is the only option that introduces credit billing for a *panel* model, i.e. the exact thing the standing rule names (BRIEF.md:84–85); B is blocked-ish by design (the custody router's key is not allowed `groq-qwen3.6-27b`, BRIEF.md:70–71) and widening key scope for an optional seat is scope creep; gemini is rule-compliant if its CLI is subscription/OAuth-authed, but it is *not authenticated* (BRIEF.md:84) and adds an unverified seat mid-interim. If the GB10 comes back for the assistant, the real Qwen seat returns free — the panel question dissolves.

```
CLAIM: Option A violates the rule's letter but not its purpose, and is acceptable as a panel third seat only if the owner explicitly time-boxes it and it is capped by a dedicated LiteLLM virtual key at ~$10/30d; absent that, C is the correct default.
EVIDENCE: BRIEF.md:145-146 "The 'no API-credit panel models' rule was a cost and auth-hygiene decision, not a data-custody one"; cost basis BRIEF.md:104 "estimated under $5/month"; cap mechanisms available: BRIEF.md:66 "max_budget: $100/30d" and BRIEF.md:135 "50 req/60 s limit and a 1 h cache".
CONFIDENCE: 0.6
FALSIFIER: Owner says the rule is about billing relationships per se (not cost/hygiene), making A a flat violation at any cap; or says <$5/mo is beneath notice and no cap is needed.
```

Auth hygiene actually cuts *for* A on one axis: a scoped `Workers AI: Read` token is strictly better than the Global key. That argues for minting it — just not as a panel-seat justification.

```
CLAIM: Each recommendation's dominant silent-failure mode is: (B) a 200 OK with empty `content` from the reasoning model, caught by the arithmetic probe at max_tokens=300 through LiteLLM with the router's own key asserting non-empty content; (assistant-down) the "for now" loan never ending, caught by a dated return deadline; (panel-C) a 2-seat verdict being read as if it were 3-seat, caught by requiring the seat roster in the panel run record.
EVIDENCE: BRIEF.md:72-73 "A probe of `groq-gpt-oss-120b` returned empty `content` at `max_tokens=20`"; BRIEF.md:131-132 "arithmetic probe per route via LiteLLM with the router's key"; BRIEF.md:61-62 "indefinite 'for now'". Secondary B mode: BRIEF.md:128-129 "risk = silent cost/quality drift if nobody notices the fallback stayed hot" — caught by checking the groq deployment's request count is 0 after `close` runs.
CONFIDENCE: 0.7
FALSIFIER: The retest returns normal content at 300 tokens AND LiteLLM surfaces empty-content responses as errors — then the dominant B mode is instead the stuck-hot fallback, not empty content.
```

```
CLAIM: Leave the assistant down and ask the partner to return the GB10 (or run `close` on the GB10 only); do not override custody — the deciding factor is that the custody override is the sole irreversible action in the decision set, while every alternative is reversible in minutes.
EVIDENCE: BRIEF.md:124-127 A1 "All reversible" for config/adapter/policy changes, versus BRIEF.md:129-130 "The custody override = client-confidential data leaving prem (contractual/legal, not technical)"; feasibility of the ask: BRIEF.md:62-63 "A `close` script restores prod at any time; timing is the operator's choice"; sufficiency: BRIEF.md:118-119 GB10 alone carries the model "at reduced quality".
CONFIDENCE: 0.75
FALSIFIER: The assistant is load-bearing for a client SLA during the interim (an outage costs more than the disclosure risk), or the client contract already permits named cloud subprocessors — either makes a scoped, time-boxed override the right call. Also falsified if the partner's benchmark specifically requires both boxes and the loan was contractually committed.
```

One dependency worth stating: the GB10 recommendation is doubly leveraged — it restores the assistant *and* the panel's Qwen seat, which is why I would put the reclaim ask ahead of any cloud work for consumers 2 and 3.

VERDICT INPUT: LiteLLM=B ; Assistant=reclaim-GB10 ; Panel-seat=C ; hard-cap=$25/30d
