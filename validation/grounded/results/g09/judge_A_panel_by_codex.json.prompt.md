You are scoring one review memo against a sealed rubric. You are not a participant; do not add your own review.
Score ONLY from the memo text. Quote the memo span that earns each point; if no span supports a point, the score is 0.

=== SEALED RUBRIC ===
# G09 sealed rubric - SEALED (arms never see this)

- **difficulty:** subtle (ten oracle-verified defects; one confident mechanism claim was withdrawn by its author)
- **correct_call:** **Not fit as it stands; decide-after-check on two items.**
  Blocking set, all verifiable in the tree: a template cloned verbatim from
  one real practice's business rules is served to every psychology prospect
  as their own (V10); any share-link holder can write into a public replay
  (V2); the post-tool follow-up completion omits `tools` and 400s at the
  lead-capture moment under the deployed `drop_params` routing (V1); the
  obvious liveness fix keys on a signal that proves nothing about init (V9);
  "practice practice" welcome copy for two verticals. Minimum: a parent
  liveness watchdog keyed on the honest init signal with origin checks,
  `tools=` on the follow-up (verified live, not against a mocked 400), fix
  the welcome copy, bind `demoToken` to the resolved share, genericize the
  cloned template. Defer: a second frontend runtime, paid monitoring SaaS,
  an iframe-less rewrite, a distributed circuit breaker, a Turnstile/trust
  redesign.
- **oracle:** every V-item below was confirmed by the orchestrator opening
  the cited span; the fixes were deployed and verified afterwards
  (`outcome_source`).
- **correct fix:** as in correct_call.
- **must_catch:**
  1. **V10 (severest):** `packages/shared/src/flow-templates.ts:356,407,427,431`
     - the `psychology-practice` template is one named practice's rules
     (a named practice's self-pay statement with two dollar amounts, and a
     two-state licensure restriction); `apps/crm/src/app/api/chat/widget/init/route.ts:144`
     deep-clones the whole flow for any demo with that templateId and
     overrides only `link` URLs (`:150-157`), then passes it into the system
     prompt (`:161-171`). Only this template is contaminated.
  2. **V2:** `init/route.ts:66` reads `demoToken` from the body; `:218` stores
     it in session metadata unconditionally (spread not gated on
     `resolvedDemo`); `apps/crm/src/lib/live-preview/demo-session.ts:7-14`
     has no token field to cross-check; `persist-finalize.ts:112-117`
     appends to the share `WHERE token = ${token}` - a share-link holder can
     poison a public replay and complete it.
  3. **V1:** `llm-pipeline.ts:99-101` passes `tools: qualifierTools`; the
     follow-up at `:190` passes only `{ model, messages, max_tokens }`;
     `context/litellm-routing-SANITIZED.yaml:42-43` `drop_params: true`
     with no `modify_params`; capture C7 shows the 400.
  4. **V9:** `apps/chat/app/components/ChatWidget.tsx:100-102` shows the
     pre-chat form and clears loading before init resolves; init runs in
     the background with its rejection swallowed at `:117-119`;
     `PreChatForm.tsx:69-71` posts `triagecrm:prechat-complete` on an
     animation timer, so a watchdog keyed on it is satisfied by a widget
     whose init hung; the honest signal is `live_preview_widget_init`
     (`ChatWidget.tsx:78`).
  5. **V6:** neither parent listener checks `e.origin`/`e.source`
     (`apps/web/src/scripts/live-preview/animation.ts:51-56`,
     `simulator-bridge.ts:92`); the fallback zoom at `animation.ts:58-62`
     zooms a dead iframe after 5 s with no error step (silent success).
  6. **"practice practice":** `init/route.ts:258-263` `buildDemoWelcome`
     appends " practice" to a template name that already ends in "Practice"
     (`flow-templates.ts:512,544`); capture C8.
  7. **V3:** `ChatWidget.tsx:224-236` posts `chat-message` then `chat-done`
     with the same payload on the final turn; the simulator reducer pushes
     both (`reducer.ts:67,119`) - duplicated closing exchange.
  8. **V4/V5:** `prefetch/route.ts:19` `SEMAPHORE_TTL_MS = 30_000` vs
     `retries: 1` x `SCREENSHOT_TIMEOUT = 20_000`
     (`screenshot-service.ts:13`, `prefetch/route.ts:76`) - up to 40 s of
     work under a 30 s lease; the unbounded wait is `arrayBuffer()` at
     `screenshot-service.ts:161`; `getDemoShare` returns the Redis-cached row
     before the expiry check (`demo-share.ts`).
  9. **C3 explanation:** a browser-side stall after the two entry chunks and
     before the widget route executed; neither Cloudflare nor the demo API
     (C4/C5); the exact exception is NOT determinable from the captures
     (C3 records requests, not JS execution), so "the router never issued
     the dynamic import" is over-claimed - `routeTree.gen.ts:15` statically
     imports the route while production serves a separate chunk, which does
     not establish why the browser never requested it. The right answer is
     to make the class loud, not to diagnose it.
  10. Silent failures with cheap detectors: the 5 s fallback zoom
      (`animation.ts:58-62`), swallowed init rejection (`ChatWidget.tsx:117-119`),
      `appendMessage` errors caught by the caller (`demo-share.ts` comment),
      no minute-scale telemetry for a degraded demo.
  11. `prefetch/route.ts:55-85` DOES call `validateNotInternal(url)` - an
      SSRF claim that ignores this is wrong; the residual is that screenshot +
      page-text fetch are reachable with no Turnstile.
- **landmine (confident wrong answers):** "TanStack router never issued the
  dynamic import" stated as fact (withdrawn under cross-examination);
  "prefetch is SSRF-open" (validated); "Redis cache is generally inconsistent"
  (`appendMessage` does `redis.del`, `demo-share.ts:48-54`; the defect is the
  expiry-skip only); "the mocked 400 test verifies the tools fix" (it tests
  the app, not the deployed provider contract); "HIPAA blocker on synthetic
  Sarah Johnson data" (demo handles synthetic patients only).
- **unverifiable:** the actual exception behind C3 in the owner's browser.
- **outcome_source:** `2026-09-13-triagecrm-demo-funnel/ledger.md` (A2-A7,
  B3, B5, B6 VERIFIED) and `verdict.md` (V1-V10 oracle-settled; B2
  overturned; surviving dissent = V3 severity and a public cost ceiling);
  fixes deployed and verified per the project memory note (2026-09-13).


=== MEMO (author unknown) ===
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


=== OUTPUT (JSON only, no prose before or after) ===
{
 "decision_correct": <0 | 0.5 | 1>,
 "decision_correct_quote": "<memo span>",
 "must_catch": [ {"item": <n>, "hit": <0|1>, "quote": "<memo span or empty>"} ... one entry per must_catch item in the rubric, in order ],
 "false_objections": [ {"quote": "<memo span>", "why": "<one clause>"} ... ],
 "notes": "<one sentence>"
}