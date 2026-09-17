# Oracles — g10 (brief-only: the only checkable pointer class is "the brief says X")

Method: every brief statement either seat relied on was searched verbatim in decisions/g10-brief.md
(brief-quotes.json holds the 30 strings and line hits). Result: 30/30 present; one ("NO
`router_settings.fallbacks` configured") spans lines 18-19 and matched on the wrapped line.
No artifact directory exists, so grounding.py was not run (not applicable).

Verified by inspection (the brief text plainly settles the factual basis):
- A2 / B2 mechanism: brief:36-37 states the rule verbatim; :44 "Needs an API token with `Workers AI: Read`";
  :43 usage-based billing; :97 the rule was "a cost and auth-hygiene decision". So Option A as a panel seat =
  API token + usage billing, which is the thing the rule names. VERIFIED (basis). The "in spirit" judgment is the seats'.
- Shared facts both cite correctly: probe empty at max_tokens=20 with retest pending (:24-25); no fallbacks configured
  (:18-19); assistant on-prem only, custody proof prunes at generation time (:29-32, :100-101); Codex+Grok verified,
  2-seat floor (:35, :66); GB10 alone could carry the assistant at reduced quality (:70-71); fallback re-takes traffic
  when the RTX returns (:79-80); Groq killed two Llama models (:61-62); cpu-limp is a stub (:28).

Not settled by the brief (PANELIST-CLAIM stays): which LiteLLM option (A vs B) — the brief gives facts for both and
states no preference; assistant default (reclaim vs down) and its deciding factor — the brief says timing is the
operator's choice and offers D as options; Q3 likelihood rankings — no failure rates in the brief; the $5 figure is
an estimate (:56), not a stated cap.
