Plausible framings: bootstrap incident diagnosis, conversion polish, or end-to-end release readiness. I choose readiness: C3 demonstrates failure before chat, while C6 demonstrates that successful chat alone does not establish the whole funnel. Capture references below are **EXTERNAL**, sourced from `artifact/context/production-captures-2026-09-12.md`; source paths are under `artifact/`.

1. **CLAIM:** C3 most likely represents client bootstrap/hydration failure before route execution, but its exact cause is unproven.
   
   **EVIDENCE:** EXTERNAL C3–C5; `apps/chat/app/client.tsx:6–8`; `ChatWidget.tsx:94–109`.
   
   **CONFIDENCE:** 0.85
   
   **FALSIFIER:** An owner-browser trace showing route execution and an init request omitted from the capture.

   No stable position on browser incompatibility, extension interference, or hydration exception. Capture the owner’s console and network trace. Add an independent parent deadline requiring a validated widget-ready message after initialization and rendering; on expiry, show a useful fallback and report failure. Install bootstrap error reporting before application imports. Disabling lazy loading alone cannot eliminate bootstrap failures.

2. **CLAIM:** Screenshot readiness has incomplete deadlines and a lease shorter than permitted retry work.
   
   **EVIDENCE:** `requestScreenshot`, `captureAndStore`; public `prefetch/route.ts:19,75–89`; web `submitForm:55–110`; `semaphore.ts:17–27`.
   
   **CONFIDENCE:** 0.96
   
   **FALSIFIER:** A higher-level enforced deadline cancels all screenshot work before lease expiry.

   A1: readiness means successful response body plus completed storage writes, not decoded browser image. There is no polling. Fetch has a 20-second timer cleared before body consumption; prefetch permits one retry after one second, exceeding its 30-second unrenewed lease. Submit has no internal retry; the browser retries selected errors after one/two seconds. Prefetch is sampled once through a one-second race. Bound the entire operation, cancel abandoned work, and align lease duration or renewal.

3. **CLAIM:** Session, share, widget, and tool readiness mostly relies on awaited calls without end-to-end deadlines or reconciliation.
   
   **EVIDENCE:** `createDemoSession`; public `route.ts:292–328`; widget `init/route.ts:113–221`; `callLLMWithTools`; `executeToolCalls`; chat `lib/api.ts:40–49`.
   
   **CONFIDENCE:** 0.94
   
   **FALSIFIER:** Supplied dependency implementations establish bounded completion and reconciliation for these callers.

   A1: demo session requires Redis SET completion, then one GET at init; TTL 30 minutes. Share requires insert plus screenshot-URL update; no polling/retry. Widget requires conversation/messages/Redis writes and client init resolution; no client timeout/retry. Each LLM call has 30 seconds, no explicit application retry; proxy configuration specifies two retries and 30 seconds (`litellm-routing-SANITIZED.yaml:55–58`). Tools execute sequentially without local timeout/retry. Underlying SDK policies are unreadable here: **SPECULATIVE**, settle with delayed-dependency probes.

4. **CLAIM:** Replay consistency and producer-side success checks can disagree with what prospects receive.
   
   **EVIDENCE:** `getDemoShare`, `appendMessage`, `completeDemoShare`; `loadDomainCache`; `startPrefetch`; public `route.ts:396–434`; screenshot route `:122`.
   
   **CONFIDENCE:** 0.96
   
   **FALSIFIER:** Concurrent read/write and invalidation-failure probes always return current replay state.

   A2: share reads cache for one hour; append invalidation errors are swallowed, completion does not invalidate, and cache hits skip expiry validation. Read-fill races can repopulate stale data. Domain caching collapses paths for seven days. No server negative cache is shown; failed browser prefetch suppresses same-URL retries. DB/blob replication lag is unproven. Outbox delivery is asynchronous (`OutboxEvent`).
   
   A3: storage acknowledgements/internal reads and screenshot-success emission do not verify public rendering. Require browser image decode, rendered chat, and a fresh-context replay check; disable sharing until these pass.

5. **CLAIM:** A run spans independently updated stores and is not idempotent across interruption.
   
   **EVIDENCE:** public `route.ts:243–383`; `storeScreenshot`, `storeMetadata`; widget `init/route.ts:110–221`; `persistAndFinalize`; web `form.ts:124–128`; `session-lock.ts:4,23–38`.
   
   **CONFIDENCE:** 0.93
   
   **FALSIFIER:** Crash/retry probes demonstrate one logical run and complete cleanup.

   A4 inventory, writer → skipped-write symptom:
   - Submit: Contact/customFields, TagAssignment → missing prospect/onboarding attribution; DemoShare → broken replay.
   - Init/message/tool paths: Conversation, ChatMessage, Redis session, Contact/Lead → missing history/state/conversion.
   - Finalize: outbox → absent downstream completion; audit/analytics → missing evidence.
   - Submit/prefetch: JPEG, metadata JSON in R2 or disk → missing image/reuse; Redis domain cache → recapture.
   - Share readers/writers: Redis share cache → stale replay.
   - Requests: Redis rate limits, semaphore, locks → lost controls.
   - Browser: prefetch promise, event buffer, React state, URL/history → stale/missing display or resume.
   - Process: circuit breaker/telemetry buffers → lost protection/evidence.
   - LiteLLM: Redis completion cache → caching behavior changes.

   Other callers of these helpers can write the same stores; external writers and `createLeadFromChat` internals are unavailable. CDN writes/purge configuration is unavailable; image policy is private/1800 seconds. A5: aborts can orphan images, sessions, conversations and partial shares; reload enters replay, not resume (`index.ts:6–9`). Add run/message idempotency keys before automatic mutation retries.

6. **CLAIM:** The post-tool completion contract explains C7 independently of C3.
   
   **EVIDENCE:** EXTERNAL C7; `llm-pipeline.ts:180–199`; `__tests__/llm-pipeline-tool-drop.test.ts:23–28`.
   
   **CONFIDENCE:** 0.98
   
   **FALSIFIER:** The captured failing request included valid tool definitions.

   Supply the tool definitions on follow-up and verify a forced tool turn through the deployed proxy. C7 establishes actual dependency behavior; the fake model does not. C6 does not establish lead creation: its captured lead is null. Alert on follow-up fallback, dropped-tool counters, and lead-save failure; assert the alert using injected failures. Do not reopen the historical SQL-column incident without recurrence evidence.

7. **CLAIM:** Public abuse and replay boundaries need targeted gates, not a speculative security redesign.
   
   **EVIDENCE:** prefetch `route.ts:42–76`; `validateNotInternal`; widget init `:134–136,218`; `appendMessage:42–46`; `persistAndFinalize:114–115`.
   
   **CONFIDENCE:** 0.91
   
   **FALSIFIER:** Integration probes demonstrate mandatory capture admission, destination isolation, and server-bound share ownership.

   Prefetch spends screenshot capacity without Turnstile; client-supplied demoToken reaches token-only updates without ownership binding. Bind it server-side. Add a global spend/concurrency budget. Screenshot-service redirect/subresource enforcement is unreadable: **SPECULATIVE**, test controlled private destinations before exposure. Free text reaches plaintext share JSON; synthetic pre-chat identity does not prevent real patient text. Disable public transcript sharing initially and label synthetic use explicitly.

8. **CLAIM:** Several degraded outcomes have no demonstrated operator notification despite existing logs and metrics.
   
   **EVIDENCE:** `animation.ts:58–62`; `fetchPageText:55–56`; `tool-executor.ts:107–123`; `persistAndFinalize:119–120`; `ChatWidget.tsx:224–235`; simulator `simReducer` MESSAGE/DONE; EXTERNAL C8–C10.
   
   **CONFIDENCE:** 0.88
   
   **FALSIFIER:** Fault injection produces timely existing operator alerts.

   Cheapest detectors: parent readiness/image deadlines; Turnstile error/expiry callbacks; counters for empty crawl, ungrounded service fallback and replay-write failure; browser assertions for duplicate final simulator messages and welcome copy. MESSAGE plus DONE currently appends final messages twice. Fix that and “practice practice.” Actual alert delivery remains unverified. Run a synthetic complete conversation/replay every few minutes with dead-man supervision independent of C9’s failing NATS loop; test detector failure too.

**VERDICT INPUT**

Q1: No; ship the bounded fixes/gates above with destination checks. Defer framework replacement, extraction redesign, paid monitoring and a second runtime.

Q2: Bootstrap failure is the strongest localization; exact cause unresolved. Parent-enforced readiness makes it visible.

Q3: Prioritize bootstrap, image/Turnstile stalls, fallback degradation, simulator corruption and replay divergence; logs alone do not establish notification.
