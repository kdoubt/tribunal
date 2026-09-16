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
