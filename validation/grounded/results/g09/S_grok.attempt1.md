I'll ground this in the actual demo funnel files and the production captures before taking a position. Alternative framings first, then a full read of the page, APIs, widget, and C1–C10.The captures and source live under `artifact/`. I’ll read C1–C10 and the demo page, widget, and API paths next.C3 points at a lazy-route-chunk miss, so I’m reading the widget SPA and the live-preview client next.C3’s missing route chunk is the main risk; I’m reading init, LLM follow-up, telemetry, and the screenshot/session path next.**Framings considered:** (1) binary ship/no-ship of the whole funnel; (2) C3 as owner-browser flake vs a systemic “success without consumer proof” bug; (3) public-endpoint abuse/HIPAA as the gate; (4) C8 copy as the only blocker. **Picked (1) constrained by (2):** fitness is whether a prospect can finish a correct demo that someone is told about. C4 shows the widget *can* work; C3 plus the parent’s 5s zoom shows a dead demo can still be labeled live.

### A. Operational readiness

**A1.** Screenshot: one `POST` to `SCREENSHOT_SERVICE_URL`, 20s abort (`screenshot-service.ts:13,79–80`); submit path `retries` default **0** (`:135`); prefetch passes `retries: 1` (`prefetch/route.ts:76`). Client retries only `TRANSIENT_CODES` twice (`form.ts:8,104–110`). Circuit: 3 fails / 30s reset / half-open 1 (`route.ts:35–38`). Semaphore: max 3, 30s TTL, fail-closed (`constants.ts:6`, `semaphore.ts:62–64`). Prefetch wait: **1s race then drop** (`form.ts:57–67`). Demo session: Redis `SET EX 1800`, read **once** at init (`demo-session.ts:5,17–19`; `init/route.ts:134–136`). Widget session / init: **once**, no retry until pre-chat submit (`ChatWidget.tsx:106–119`). LLM: 30s abort (`llm-pipeline.ts:95–108,187–195`); LiteLLM `timeout: 30`, `num_retries: 2` (`litellm-routing-SANITIZED.yaml:57–58`). Tool follow-up: one call, catch → fallback (`llm-pipeline.ts:197–199`). Page-text: 10s, `''` on fail (`page-text.ts:18`). Turnstile: 5s, `false` on fail (`trust.ts:47–67`). **Everything async is probed once** except screenshot client retry and prefetch’s single retry.

**A2.** Redis domain cache 7d with blob existence check (`domain-cache.ts:21,54–66`). Prefetch meta TTL 10 min (`constants.ts:7`, `route.ts:481`). DemoShare Redis 1h; `appendMessage` DELs it (`demo-share.ts:6,48–54`) — GET share can be stale if DEL fails. LiteLLM `cache: true` Redis (`litellm-routing-SANITIZED.yaml:44–52`). In-process circuit breaker. `history.replaceState(?d=)` before any chat (`form.ts:125–128`). Negative: prefetch miss/URL mismatch (`route.ts:479`); trust reject; prefetch `.catch(() => null)` (`prefetch.ts:37`).

**A3.** Parent treats **its own** `POST /live-preview` 200 as “chatbot is live” (`live-preview.astro:124–125`, `form.ts:149–152`) and never waits for `_token` chunk, `POST .../init`, or `triagecrm:prechat-complete`. Screenshot GET **is** independent (C3 got 200). Replace with: iframe `postMessage` (or nginx: widget GET then `_token` + init).

**A4.** Screenshot blob (R2/disk) — skip: broken img. Prefetch metadata — skip: recapture. Redis `live-preview:domain` — skip: slower recapture. Redis `demo:session` — skip: `resolvedDemo` null, generic bot. Redis semaphore — skip: 503 `BUSY`. Prisma `Contact`/`TagAssignment` — skip: no prospect row. Prisma `DemoShare` — skip: 500 now. Prisma `Conversation`/`ChatMessage` — skip: no chat. Chat session store — skip: later `SESSION_EXPIRED`. `DemoShare.messages`/`crmSnapshots` — skip: empty replay (`persist-finalize.ts:119–121`). Browser `?d=`. In-process circuit. CrmSimulator React state. PostHog. LiteLLM Redis cache.

**A5.** Close after POST: Contact, DemoShare, blob, 30-min session orphaned; `?d=` is **replay**, not live resume. Contact upsert by `emailHash` is idempotent (`route.ts:244–247`); widget init is **not** (new conversation). Restart: circuit resets; semaphore holders expire 30s; Redis sessions survive.

---

```
CLAIM: The funnel is not fit for prospects: a generated preview can be shown as live with a non-interactive widget, plus doubled welcome copy and a silent post-tool LLM failure.
EVIDENCE: EXTERNAL C3 (widget 200, no `_token` chunk, no init) + live-preview.astro:124 “Your AI chatbot is live!” set at form.ts:149–152 before any widget signal; C8; C7
CONFIDENCE: 0.88
FALSIFIER: A production owner-browser run that requests `_token-*.js`, issues init 200, and renders chat without the doubled “practice” string or `LLM follow-up call failed`
```

```
CLAIM: C3 is the CSR widget shell (1293 B) plus two entry modules loading, while ChatWidget/`POST .../init` live in a lazy `/widget/$token` chunk the HTML parser never referenced, so that `import()` never ran.
EVIDENCE: EXTERNAL C3 vs C4 (`_token-gQRoBIhn-c32dea4.js` then init 200); ChatWidget only imported from `$token.tsx:3`; init only from ChatWidget.tsx:106
CONFIDENCE: 0.78
FALSIFIER: Built widget HTML contains a parser-visible script/modulepreload of `_token-gQRoBIhn-c32dea4.js`, or C3’s IP shows that GET
```

```
CLAIM: Why that `import()` did not run in the owner’s browser (extension, hydration throw, match miss) is unknown; C4(b) rules out “iframe/sandbox always blocks.”
EVIDENCE: EXTERNAL C4(b) Playwright set `#preview-widget` src and got `_token` + init; apps/chat has no `errorElement`/`ErrorBoundary`; SPECULATIVE without owner UA/console
CONFIDENCE: 0.70
FALSIFIER: Owner DevTools showing `_token` requested or a hydration exception in Sentry for that interval
```

```
CLAIM: The C3 class becomes visible with a parent watchdog, and the “chunk never requested” signature becomes impossible if `/widget/$token` is eager in the already-fetched ~98KB client bundle (or modulepreloaded).
EVIDENCE: animation.ts:58–62 `setTimeout(..., 5000)` calls `triggerZoom()` with no error/telemetry when `triagecrm:prechat-complete` is absent; PreChatForm.tsx:69–70 is the only emitter of that message
CONFIDENCE: 0.86
FALSIFIER: After eager-import/modulepreload, a C3-style nginx trace still lacks `_token`; or a broken iframe still reaches `step-preview` without a PostHog/Sentry timeout
```

```
CLAIM: Demo welcome always appends “ practice” to the template name, so `psychology-practice` → “psychology practice practice.”
EVIDENCE: init/route.ts:261 `I detected you're a ${templateName.toLowerCase()} practice`; flow-templates.ts:543–544 `name: 'Psychology Practice'`; EXTERNAL C8
CONFIDENCE: 0.96
FALSIFIER: Live transcript for that template without the doubled word, or `name` no longer containing “Practice”
```

```
CLAIM: After a native tool call, the follow-up completion omits `tools`, which Anthropic/LiteLLM 400s; the widget still 200s a canned fallback — tests fake this call and cannot catch it.
EVIDENCE: llm-pipeline.ts:189–199 `{ model, messages, max_tokens }` with no `tools`; EXTERNAL C7 `UnsupportedParamsError` / `Available Model Group Fallbacks=None`; litellm-routing-SANITIZED.yaml:43 `drop_params: true` and no `modify_params`; route.test.ts:326–329 mock follow-up
CONFIDENCE: 0.90
FALSIFIER: A tool-using live turn with no `LLM follow-up call failed` and a non-fallback model reply
```

```
CLAIM: Silent prospect-facing failures and cheapest detectors: (1) widget GET without `_token` or init — nginx dead-man 15s + parent timeout event; (2) C7 follow-up — grep `LLM follow-up call failed`; (3) share JSON append — grep `[demo-share] Failed to record message` or share GET `messages: 0` after a 200 chat; (4) PostHog `screenshot_success` without `live_preview_widget_init`.
EVIDENCE: persist-finalize.ts:119–121 catch; ChatWidget.tsx:78 `live_preview_widget_init` only after init; live-preview/route.ts:406 `LIVE_PREVIEW_SCREENSHOT_SUCCESS` on POST 200; EXTERNAL C2/C3 log method
CONFIDENCE: 0.84
FALSIFIER: Those four signals already alerting on the app host for 2026-09-12 17:59–18:05 UTC
```

```
CLAIM: Do not do now: second chat runtime, screenshot/semaphore/circuit rewrite (C3 screenshot 200), paid APM, demo-PHI hardening, or TanStack replacement; abuse controls already fail closed (Turnstile/trust 400s are loud).
EVIDENCE: EXTERNAL C3 screenshot 200; C10 `TURNSTILE_REQUIRED` / `missing_timing`; B (workers/chatbot-api demoted); criterion 4–5
CONFIDENCE: 0.82
FALSIFIER: A prospect-visible dead demo caused by screenshot/semaphore rather than widget hydration, or real patient PHI reaching `/api/public/live-preview`
```

**VERDICT INPUT**

- **Q1:** No. Minimum: parent watchdog (no `triagecrm:prechat-complete` → error + PostHog/Sentry, not zoom); eager-load or modulepreload `_token`; fix `buildDemoWelcome`; pass `tools` on the follow-up completion; nginx `widget GET` without init. Not now: second runtime, capture-pipeline rewrite, paid SaaS, HIPAA demo-PHI, SPA rewrite.
- **Q2:** Lazy `$token` chunk never `import()`’d after the 1293 B shell + entry chunks; ChatWidget/init never mounted. C4 shows the same URL works when JS reaches that import. Make it visible (watchdog) and stop the chunk-miss (eager/modulepreload).
- **Q3:** C3-class dead iframe (nginx 15s + parent timeout); C7 follow-up (`LLM follow-up call failed`); empty replay after chat (`[demo-share]` / share GET); screenshot_success without widget_init (PostHog). C8 is silent wrong copy — unit-test the welcome string.
