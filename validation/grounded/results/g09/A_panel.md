# Verdict - g09: is the public live-preview demo funnel fit to put in front of prospects?

Mechanical fill from ledger rows only. Seats A/B; rounds R0+R1.

## 1. Independent agreement (agreed-r0)

The panel concludes:
- The funnel is not fit as it stands (A1, B1).
- C3's class: the client graph never reached the lazily imported `/widget/$token` route, so ChatWidget never mounted and `init` was never issued; the browser-level trigger is not determinable from the tree (A2, B2, B3).
- The parent page's unconditional 5-second zoom (animation.ts:58-62) labels a dead widget "live"; it must become a watchdog that errors and reports (A1, B4 watchdog half).
- The post-tool follow-up completion omits `tools`, which the deployed LiteLLM config (drop_params, no modify_params) rejects for the Anthropic deployment; the failure is swallowed into canned copy (A4, B6; C7).
- The welcome string doubles "practice" (A7, B5; C8).

## 2. Resolved after Round 0

**Oracle-settled (`verified`):** every premise above by inspection (see oracles.md); A3 token length within bound; A5's premise (two triagecrm-chat deployments under latency routing); A6's premises (build-time sitekey vs runtime secret, synchronous token read, TURNSTILE_REQUIRED non-retryable); A8's premise (in-process telemetry buffer, no caller of getMetricsSummary) and B7's PostHog events (both exist); persist-finalize swallows share-append failures; the follow-up completion is mocked in tests; init/route.ts:27 re-reads the demo session (settles B's A-section correction); sentry-init returns early without a DSN.

**Cross-examination-settled (`conceded`):**
- A conceded: PreChatForm.tsx:69-70 is the sole emitter of prechat-complete; C4(b) excludes blanket iframe blocking; Sentry is in the entry graph, so "structurally unobservable" was too strong (A2 0.75→0.60); share-append swallow belongs in the silent list; A8 "no shipped observability" overstated (PostHog events exist; the missing piece is an alert).
- B conceded: A3 is true (not causal); the Turnstile build/runtime split is a deploy footgun; getMetricsSummary has no caller; screenshot POST is probe-once with retries 0; iframe src assignment is self-confirmation; share insert-then-update is two-step; resume is not idempotent; replay img.src has no onerror against a 30-day share TTL and a .tmp fallback path (B7 revised to add it, 0.84→0.86).
- Refinements accepted by both: the watchdog's 5 s budget spans the init round-trip, so it can misfire on a healthy slow run (A1 0.92→0.93); B2 restated as "graph never resolved the lazy route" (0.78→0.84).

## 3. Surviving dissent

- **Turnstile in the minimum (A6 vs B8).** A: the render/await/retry-code trio blocks real prospects (C10: hosts that cannot reach the challenge "unable to complete the form at all", retry button hidden) - criterion-1 damage fixable in hours. B: the gate is loud and fail-closed, did not cause C3, and no evidence shows sitekey and secret disagree - not min-fit. Cheapest discriminating test: submit the real form from a browser with the challenge host blocked and record what the prospect sees and whether a retry is offered.
- **Eager-import remedy (B4 vs A2).** B: eager-importing or modulepreloading `$token` makes the "chunk never requested" signature impossible. A: C3 shows the entry chunk was fetched, not executed; eager import removes the signature but not the failure, and a watchdog keyed on prechat-complete false-positives on slow init - key it on a distinct widget-ready at init success. Cheapest test: build with the route eager, reproduce a hydration throw before route resolution, and observe whether init still fails with no signature.
- **C7 intermittency (A5).** A: latency routing across two deployments. B: routing predicts stickiness; run 2 may have had no tool call; SPECULATIVE without LiteLLM logs. Cheapest test: read the LiteLLM proxy log for the two C6 runs.
- **Detector cheapness (B7 vs A).** B's grep and PostHog-funnel detectors vs A's per-session server-side 90 s init dead-man (funnel-rate alarms have no base rate at ~3 events per 15 days). Test: count how many of the four detectors would have fired within minutes on the C3 window at that traffic.

## Recommendation

**Mode:** `dont` (as it stands) → ship after the agreed minimum.

The panel concludes (bucket 1) the funnel is not fit and that the agreed minimum is: a parent watchdog that shows an error and emits an event instead of zooming when the widget never signals (A1/B4), `tools` on the follow-up completion (A4/B6), and the welcome-string fix (A7/B5). Resolved items add a share/replay image onerror and a Sentry-DSN-at-build check (bucket 2). Whether Turnstile's render/await/retry defects (A6/B8) and eager-loading the route (B4/A2) belong in the minimum, which detector is cheapest at this traffic, and what explains C7's intermittency, remain surviving dissent (bucket 3) for the human, each with its test named.

## Record

- Open (unexamined, not endorsed): none.
- Verified vs merely agreed: verified = A1, A2 mechanism, A3, A4, A6 premises, A7 (psychology), A8 premise, B1, B2, B4 watchdog premise, B5, B6, B7 mechanisms; agreed-only = B3/A2 trigger unknown, the "not fit" headline.
- Rounds run / stop rule hit: R0+R1; stopped: no load-bearing claim flipped, no new claim IDs.
- Transformations applied to relayed text: neutral Peer labels, shuffled order, ORCH-SUMMARY note; nothing else.

VERDICT INPUT: Q1 - not fit; minimum = parent watchdog (error + event, no blind zoom), `tools` on the follow-up completion, welcome-string fix, plus share-image onerror; Turnstile trio and eager route import = surviving dissent for the owner; not now = second runtime, screenshot/semaphore/breaker rewrite, paid APM, demo-PHI hardening, TanStack swap. Q2 - the client graph never reached the lazy route, so no chunk request and no init; trigger undetermined; make it visible with the watchdog (both) and a server-side init dead-man (A); eager import: narrows (A) vs eliminates (B). Q3 - silent: dead widget under a "live" heading, C7 follow-up swallow, share-append swallow, expired/missing replay screenshot with no onerror, screenshot_success without widget_init; cheapest detectors disputed between per-session dead-man (A) and grep/PostHog-ratio (B).
