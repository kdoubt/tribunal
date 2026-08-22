DECISION MEMO — C18 Feature flags: build vs buy

A

A 25-engineer org that wants flags, percentage rollouts, *and* basic experiments is buying a product, not a config table. Targeting, sticky assignment, kill switches, audit, multi-language SDKs, streaming/local eval, and even crude experiment analysis are a standing platform. Building that “lightweight” service is 1–2 engineers forever plus a second product (experimentation) that will never get stats, assignment hygiene, or analysis right. Buy ships in days; build ships a boolean file and then a two-year LaunchDarkly-lite. At this headcount the opportunity cost of a home-grown flags team exceeds a managed seat bill. Use local evaluation and fail-open defaults so the vendor is not in the request path.

Tradeoffs
- Speed vs control: buy is live this sprint; build owns the data model and never pays MAU tax.
- Completeness vs simplicity: managed targeting/experiments/SDKs vs a YAML file that will accrete rules until it is a worse LaunchDarkly.
- Cash vs people: subscription scales with usage; build scales with paging, SDK drift, and “just one more targeting rule.”
- Dependency: vendor outage/pricing/lock-in vs an internal SPOF no one wants to own.

Single biggest risk
Vendor lock-in plus production coupling: evaluation in the request path, fail-closed defaults, and a flag graph that cannot be exported — so a price hike or a 20-minute outage becomes an incident and a multi-quarter rewrite.

Cheapest test they chose wrong
90-day kill criterion: if (a) annualized spend > ~0.3 FTE *or* a vendor blip pages on-call because eval is not local/fail-open, *and* (b) they have no live experiment and only boolean/% flags with no targeting/audit in real use — they overbought. Dump to self-hosted Unleash/Flagsmith (or a config service) before the SDK surface hardens.
