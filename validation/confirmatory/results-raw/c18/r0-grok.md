**CLAIM 1:** “Flags + gradual rollouts + basic experimentation” is a product, not a config file.
**EVIDENCE:** Percentage rollouts need stable hashing, targeting, env isolation, and fail-open/fail-closed semantics; experiments need assignment, exposure logging, and a metric path. A “lightweight SDK + JSON” almost always grows into Unleash/Flagsmith-shaped work (dashboard, audit, multi-runtime SDKs, streaming).
**CONFIDENCE:** 0.82
**FALSIFIER:** Their next 6 months of use cases are boolean/env flags plus one % rollout in a single runtime, with no experiment readout.

**CLAIM 2:** At 25 engineers, buy is cheaper than a real in-house service.
**EVIDENCE:** A credible in-house system is ~1–2 engineers for a quarter plus ongoing SDK/on-call/dashboard work. That is a large slice of a 25-person org. Managed flags at this size are typically well under one senior FTE/year unless they pick enterprise list price and over-provision seats.
**CONFIDENCE:** 0.78
**FALSIFIER:** A one-week spike ships all three needs (flag, % rollout, one A/B with a metric) across every runtime they ship, with no dedicated owner afterward.

**CLAIM 3:** Experimentation is the build-killer.
**EVIDENCE:** Homegrown A/B routinely fails on sticky assignment, exposure vs. intent-to-treat, peeking, and “we looked at revenue in the dashboard.” That is why GrowthBook/PostHog/LD Experiments exist. If they want “basic experimentation,” they are buying stats and assignment discipline, not a toggle.
**CONFIDENCE:** 0.80
**FALSIFIER:** They already have a trustworthy assignment + metric pipeline (e.g. warehouse exposures + a stats library they actually use) and only need a targeting layer.

**CLAIM 4:** The live risk of A is over-buying, not “SaaS vs YAML.”
**EVIDENCE:** LaunchDarkly-class contracts and seat/MAU pricing are where 25-person teams get burned. ConfigCat, GrowthBook Cloud, PostHog, Flagsmith/Unleash cloud cover this brief. Buy still wins; the vendor SKU is the variable.
**CONFIDENCE:** 0.74
**FALSIFIER:** The cheapest managed option that covers SDKs + % rollout + one experiment is >1 engineer-month/year *and* they have ≤2 runtimes.

VERDICT: A — Biggest risk: locking into LaunchDarkly-class price/complexity for boolean+% use. Cheapest test: 90 days of usage. If they never run an experiment, flags stay env-level on/off, and the invoice exceeds ~one senior engineer-week, they overbought (or bought the wrong vendor).
