CLAIM: A minimizes operational and organizational burden for a 25-engineer team.  
EVIDENCE: Reliable flagging requires low-latency evaluation, cached/offline behavior, targeting, audit logs, permissions, change history, observability, and safe failure modes—not merely config plus an SDK. Building and maintaining these diverts scarce engineering capacity from product work.  
CONFIDENCE: High  
FALSIFIER: The organization already operates a proven internal configuration/control-plane system satisfying these requirements with negligible incremental ownership.

CLAIM: Managed tooling better supports gradual rollout and experimentation safely.  
EVIDENCE: Mature platforms provide percentage rollouts, stable bucketing, segmentation, metrics integrations, approval workflows, and kill switches. Homegrown implementations commonly underestimate identity consistency, statistical correctness, and operational UX.  
CONFIDENCE: High  
FALSIFIER: “Basic experimentation” means only deterministic cohort assignment with analysis handled elsewhere, and advanced governance is explicitly unnecessary.

CLAIM: Buying is likely cheaper in total cost despite recurring fees.  
EVIDENCE: Even a fraction of one engineer’s annual time—plus incidents, SDK upkeep, dashboards, security reviews, and support—can exceed managed-platform pricing at this organization’s scale.  
CONFIDENCE: Medium-high  
FALSIFIER: Vendor pricing based on seats, contexts, or events materially exceeds the fully loaded cost of roughly 0.25–0.5 engineer-years annually.

CLAIM: Vendor lock-in is manageable through application-level abstraction.  
EVIDENCE: A narrow internal flag interface, standardized evaluation context, and disciplined flag cleanup preserve migration options without recreating the platform.  
CONFIDENCE: Medium  
FALSIFIER: Required vendor-specific experimentation or targeting semantics become deeply embedded across services and clients.

VERDICT: A — Biggest risk: unpredictable vendor cost and lock-in as usage grows. Cheapest discriminating test: obtain a three-year quote using projected seats, contexts/events, environments, and growth, then compare it with a realistic internal estimate including at least 0.5 engineer-year to build and 0.25 engineer-year annually to operate. Choose B only if the managed cost is materially higher and a two-week prototype proves deterministic rollout, cached failure behavior, auditability, and metric attribution with acceptable maintenance.
