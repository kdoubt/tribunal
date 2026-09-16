# Decision G10 - an interim LLM backstop while both estate GPUs are lent to a partner

Brief only: there is no artifact directory for this decision. Everything you
need is below; cite mechanisms and the stated facts.

## Situation (frozen facts)

- The estate has two GPUs: an RTX Pro 6000 96 GB (serves `gpt-oss-120b` via
  vLLM :8000, plus a broken embed service) and a GB10 128 GB unified-memory
  box (serves `qwen3-30b-a3b` :8200 = the estate's heterogeneous review-panel
  seat, plus rerank/embed).
- Both GPUs were handed, interactively and for an indefinite "for now", to an
  external benchmarking partner. Production vLLM on both is STOPPED. A `close`
  script restores prod at any time; timing is the operator's choice, hours or
  days.
- Consumers now broken:
  1. The self-hosted LiteLLM alias `gpt-oss-120b` -> connection error.
     LiteLLM (`max_budget: $100/30d`, Redis cache, analytics logging) has NO
     `router_settings.fallbacks` configured. It already carries
     `groq-gpt-oss-120b` (same open-weights model, via a Cloudflare AI
     Gateway, Groq API key already present = Groq billing already accepted)
     and `groq-qwen3.6-27b`. Virtual keys are model-scoped; the custody
     router's key is NOT allowed `groq-qwen3.6-27b`. A probe of
     `groq-gpt-oss-120b` returned empty `content` at `max_tokens=20`
     (reasoning model; a retest at 300 is pending).
  2. The custody-enforcing router's assistant route (an internal
     knowledge assistant over client email/tickets) has ladder
     `rtx-fast -> cpu-limp`; `cpu-limp` is a code stub. The route is
     `client_confidential` / on-prem only by custody policy (a custody proof
     prunes off-prem rungs at policy-generation time; an architecture
     decision record says so). So that assistant is DOWN and policy forbids
     substituting any cloud model unless the owner overrides custody.
  3. Review panels: normally Codex (OpenAI, subscription CLI) + Grok (xAI,
     subscription CLI) + an optional local Qwen seat (GB10). The Qwen seat is
     DOWN. Codex and Grok are up and verified. A Gemini CLI is installed but
     NOT authenticated. Owner's standing rule: "no API-key/credit billing for
     panel models; use subscription-auth CLIs."
- Cloudflare Workers AI (the owner asked "can we use Cloudflare LLM models in
  the meantime?"): the account catalog includes `@cf/openai/gpt-oss-120b`
  (128k ctx, same model as the RTX) and `@cf/qwen/qwen3-30b-a3b-fp8` (32k,
  same model as the GB10 seat), plus other families. Workers AI is
  OpenAI-compatible and reachable via the existing AI Gateway. Billing:
  usage-based, 10,000 neurons/day free then $0.011/1k neurons (gpt-oss-120b
  about $0.35/M in, $0.75/M out). Needs an API token with `Workers AI: Read`.
  Only the account Global API Key exists, and it LEAKED into a transcript
  recently (roll pending); a scoped token can be minted via API (precedent
  exists in another project) or in the dashboard. Secrets live in a
  self-hosted secrets store.

## Options under review

- **A. Workers AI, same models:** add LiteLLM deployments `cf-gpt-oss-120b`
  and `cf-qwen3-30b`; set `router_settings.fallbacks: gpt-oss-120b ->
  [cf-gpt-oss-120b]`; point the panel local-seat adapter at `cf-qwen3-30b`.
  Requires a new scoped CF token (mint, store), accepting usage billing
  (estimated under $5/month at current volumes).
- **B. Groq, already wired:** `fallbacks: gpt-oss-120b -> [groq-gpt-oss-120b]`;
  panel third seat = `groq-qwen3.6-27b` (Qwen family, heterogeneous).
  Requires widening one virtual key's model list and verifying the
  empty-content probe was just reasoning-token exhaustion. No new
  credentials, no new billing relationship. Groq history: it recently killed
  two Llama models the config still references (dead aliases in several
  consumers) - model-availability risk.
- **C. Do nothing cloud-side:** the LiteLLM alias stays dead; consumers
  already have Claude/GPT/Gemini aliases in LiteLLM if they choose; panels
  run 2-seat (Codex + Grok, the documented floor); the assistant stays down
  until the GPUs return.
- **D. For the assistant specifically:** an owner-approved temporary custody
  override allowing an off-prem rung, vs leaving it down, vs asking the
  partner to give the GB10 back first (the GB10 alone could carry the
  assistant's model at reduced quality).
Options for (1)/(2)/(3) may differ; recommend per consumer.

## Operational readiness (A1-A5)

- A1 Rollback: LiteLLM change = config lines + container restart (<2 min);
  seat adapter = endpoint/model constants; custody override = policy
  regeneration + router restart (fail-closed custody proven). All reversible.
- A2 Blast radius: a LiteLLM fallback triggers only on failure, so when the
  RTX returns it re-takes traffic automatically; risk = silent cost/quality
  drift if nobody notices the fallback stayed hot. The custody override =
  client-confidential data leaving prem (contractual/legal, not technical).
- A3 Verification: arithmetic probe per route via LiteLLM with the router's
  key; the panel seat = run the adapter against a stored brief; the assistant
  = one end-to-end chat.
- A4 Timing: GPUs may return any time; the interim could last days. The AI
  Gateway has a 50 req/60 s limit and a 1 h cache configured.
- A5 Owner: the estate owner executes/approves; an agent session implements.

## Precedent

- Two vLLMs on one card with GPU fractions summing above 1.0 crashlooped;
  "active is not healthy; always set start limits".
- Groq removed Llama models -> dead aliases still referenced by several
  consumers -> silent failures. Third-party catalogs churn; probe before
  relying.
- The "no API-credit panel models" rule was a cost and auth-hygiene decision,
  not a data-custody one.
- The custody proof prunes on-prem-only routes at config-generation time; a
  runtime "just this once" is architecturally impossible without regenerating
  policy (by design).

## Questions

- **Q1.** Per consumer (LiteLLM alias / the assistant via the custody router /
  the panel seat): which option, and why not the others?
- **Q2.** Does Option A violate the owner's "no API-credit billing for panel
  models" rule in spirit? If used only as a panel third seat while the GB10 is
  lent, is that acceptable, and what cap makes it safe?
- **Q3.** What is the single most likely way each recommended option fails
  silently, and the one check that catches it?
- **Q4.** Should the assistant be left down, overridden to cloud, or should
  the partner hand the GB10 back? State the deciding factor.

## Output contract

For each of Q1-Q4 produce one block:

```
CLAIM: <one sentence>
EVIDENCE: <mechanism or brief fact you rely on; ASSUMPTION or SPECULATIVE where it is not in the brief>
CONFIDENCE: <0-1 probability, calibrated>
FALSIFIER: <what observation would prove you wrong>
```

Finish with exactly one line:
`VERDICT INPUT: LiteLLM=<A|B|C> ; Assistant=<down|override|reclaim-GB10> ; Panel-seat=<A|B|C|gemini> ; hard-cap=<$ or none>`
Maximum 1200 words.
