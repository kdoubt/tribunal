A_2seat = union of two isolated Round 0 memos, no ledger, no Round 1

=== Memo 1 ===

## Framing

Three readings were available: (a) root-cause C3 only; (b) treat the funnel as a 6-hop chain and ask whether *any* hop's failure is visible; (c) argue the demo doesn't matter because C2 shows zero real traffic in 14 days. I take (b): the owner's top two criteria are "never a dead demo" and "silent failure is worse", which are properties of the chain's detectors, not of one bug. (c) is noted as a scope caveat, not an answer to the brief.

---

**CLAIM:** The funnel is not fit as-is: a widget that fails to boot is rendered as a *successful* demo, because the parent page's only post-src behaviour is an unconditional 5-second zoom, and nothing in the widget ever signals "ready".
**EVIDENCE:** `apps/web/src/scripts/live-preview/animation.ts:58-62` — "Fallback: if the iframe never sends the message (e.g. load failure), zoom after 5s" → `triggerZoom()`; the widget's earliest `window.parent.postMessage` is in `handlePreChatSubmit` (`apps/chat/app/components/ChatWidget.tsx:172`), i.e. only *after* boot succeeds.
**CONFIDENCE:** 0.92
**FALSIFIER:** A code path that hides the iframe / shows `step-error` when no `triagecrm:prechat-complete` arrives.

**CLAIM (Q2):** C3's class is "the widget's client module graph never executed to route resolution", and it is structurally unobservable because the route chunk is requested *only* as a consequence of that execution and Sentry is initialised inside the very graph that failed.
**EVIDENCE:** `apps/chat/app/routes/__root.tsx:3,8` (`import { initSentry }` … `initSentry()` at module scope); `apps/chat/app/client.tsx:6-8` (`hydrateRoot` is the only thing that resolves `/widget/$token` → its lazy chunk). The exact browser-level trigger is **SPECULATIVE** — not determinable from the tree; C5 rules out the edge, C4 rules out the URL. Live probe: load the same widget URL in the owner's actual browser with devtools open and read the console/`window.onerror`.
**CONFIDENCE:** 0.75 (unobservability), 0.3 (any specific trigger)
**FALSIFIER:** A `<link rel="modulepreload">` for `_token-*.js` in the served widget HTML — then the chunk would be fetched independent of JS, and non-execution would not explain C3.

**CLAIM:** Search-param validation is *not* the cause: `demoToken` is exactly 20 chars, within the schema bound.
**EVIDENCE:** `demo-share.ts:20` `randomBytes(15).toString('base64url').slice(0, 20)` vs `routes/widget/$token.tsx:8` `demoToken: z.string().max(20)`.
**CONFIDENCE:** 0.88
**FALSIFIER:** A deployed token longer than 20 chars.

**CLAIM:** The post-tool follow-up LLM call omits `tools`, which LiteLLM rejects for Anthropic deployments, so the demo's money moment (tool call → contact/lead) degrades to canned copy with only a `console.error`.
**EVIDENCE:** `llm-pipeline.ts:190` `{ model: MODELS.CHAT, messages: followUpMessages, max_tokens: 1024 }` (no `tools`) vs `:100` which passes them; `context/litellm-routing-SANITIZED.yaml:42-52` has `drop_params: true` and **no** `modify_params`; C7 verbatim error. Swallow at `llm-pipeline.ts:197-200`.
**CONFIDENCE:** 0.9
**FALSIFIER:** Repro with a forced `check_service` call that returns a real follow-up completion with `tools` still absent.

**CLAIM:** The intermittency of C7 (run 1 fails, run 2 clean) is explained by `triagecrm-chat` being a two-deployment group under latency-based routing, so demo replies come non-deterministically from Anthropic Haiku 4.5 or Cloudflare llama-3.1-70b.
**EVIDENCE:** `litellm-routing-SANITIZED.yaml:11-13` and `:25-27` both `model_name: "triagecrm-chat"`; `:56` `routing_strategy: "latency-based-routing"`.
**CONFIDENCE:** 0.62
**FALSIFIER:** LiteLLM logs showing both C6 runs hit the Anthropic deployment.

**CLAIM:** Turnstile is a live funnel-killer with two independent defects: the gate is enabled by a *server* env var but rendered from a *build-time* client var, and the client reads the token synchronously without waiting for the challenge.
**EVIDENCE:** `live-preview.astro:98` `{import.meta.env.PUBLIC_TURNSTILE_SITE_KEY && (...)}` (Astro build-time) vs `route.ts:133` `!!process.env.TURNSTILE_SECRET_KEY` (runtime); `form.ts:83-86` reads `[name="cf-turnstile-response"]` at submit with no await; `TURNSTILE_REQUIRED` is absent from `TRANSIENT_CODES` (`form.ts:8`), so the error is shown with the retry button hidden (`form.ts:31-35`). Corroborated by C10.
**CONFIDENCE:** 0.8
**FALSIFIER:** `TURNSTILE_SECRET_KEY` unset in production, plus Turnstile's implicit render proven to populate the field before any human can click submit.

**CLAIM:** The demo's first sentence is grammatically broken for all six verticals.
**EVIDENCE:** `init/route.ts:261` `` `…a ${templateName.toLowerCase()} practice…` `` with `flow-templates.ts:544` `name: 'Psychology Practice'` → "a psychology practice practice" (C8); also yields "a med spa practice", "a therapy & counseling practice".
**CONFIDENCE:** 0.95
**FALSIFIER:** A separate display-name map stripping the suffix.

**CLAIM:** There is no shipped observability for this funnel: the live-preview telemetry module is an in-process ring buffer with no exporter, so nothing correlates "preview 200" with "widget init".
**EVIDENCE:** `lib/live-preview/telemetry.ts:15-17,54-58` — `const metrics: Metric[]`, `console.log(JSON.stringify(...))`, `getMetricsSummary()` exported to nothing. C2 (3 preview POSTs vs 1 init POST) is the ratio no one was watching.
**CONFIDENCE:** 0.85
**FALSIFIER:** A scrape/flush of `getMetricsSummary()` or a PostHog funnel alert already configured on `LIVE_PREVIEW_SCREENSHOT_SUCCESS` → `CHAT_WIDGET_INIT`.

---

### Mandatory A-section (compressed)

**A1.** Screenshot: HTTP POST, 20 s abort, `retries ?? 0` (route.ts passes none → **probed once**), circuit breaker 3/30 s. Demo session: Redis `SET EX 1800`, never re-read before the widget loads. DemoShare: PG insert then a *second* `update` for the signed URL (`route.ts:325`) — two-step, not atomic. Widget session/LLM: 30 s `withAbortableTimeout`, no retry (`llm-pipeline.ts:107,194`). **Everything async here is probed once; nothing polls.**
**A2.** Redis demo-session (30 min) and `demo:share:` cache (1 h, invalidated by `appendMessage`) are the eventually-consistent writes; `getDemoShare` reads Redis-first and will serve a stale row if a `$executeRaw` append succeeded but `redis.del` threw (`demo-share.ts:49-54`). No negative caching. `loadDomainCache` *does* re-verify blob existence (`domain-cache.ts:64`) — good.
**A3.** Self-confirmation: the server returns `widgetUrl` and the client treats assignment of `iframe.src` as proof the widget is live (`form.ts:151-152`). Independent check: require a `triagecrm:widget-ready` postMessage, and server-side require a `chat/widget/init` bearing that `demoSession` within 90 s.
**A4.** Stores per run: Redis (`demo:session:`, `demo:share:`, `live-preview:domain:`, rate-limit, semaphore, LiteLLM cache), PG (`contacts`, `tag_assignments`, `demo_shares`, `conversations`, `chat_messages`), screenshot blob (R2 **or** `process.cwd()/.tmp/screenshots`, `constants.ts:3`), browser history (`history.replaceState`, `form.ts:128`), CRM simulator in-memory reducer. Note `demo-share.ts:7` `EXPIRY_DAYS = 30` against a `.tmp` blob path: if R2 is unconfigured, every share link outlives its screenshot, and `form.ts:349` sets `img.src` with no `onerror` → silent broken image.
**A5.** Resume is **not** idempotent: an abort after `createDemoSession` leaves a Redis session, a `contacts` row, a `demo_shares` row and (if init ran) a `conversations` + 2 `chat_messages` rows with no conversation; only the DemoShare has cleanup (`route.ts:346-360`). The semaphore self-heals via TTL (`semaphore.ts:17`).

---

**VERDICT INPUT**

- **Q1.** Not fit. Minimum: (1) pass `tools: qualifierTools, tool_choice:'none'` at `llm-pipeline.ts:190`; (2) widget posts `triagecrm:widget-ready` on mount and the parent shows `step-error` instead of zooming when it doesn't arrive in ~6 s; (3) a dead-man check that preview-success without a matching `chat/widget/init` in 90 s fires an alert; (4) delete the duplicated word at `init/route.ts:261`; (5) block submit until the Turnstile token exists and assert sitekey/secret co-configuration at boot. Not worth doing now: second runtime, PHI hardening of synthetic demo data, semaphore/eventual-consistency redesign, abuse limits beyond the existing 20/hr.
- **Q2.** The widget's module graph did not execute to route resolution in that browser; the specific trigger is not determinable from this tree, and it is undetectable because the route chunk load and the Sentry initialisation both live *inside* the failed graph. Make it visible with a classic (non-module) inline boot beacon in the widget document plus the parent handshake in Q1(2).
- **Q3.** Silent: widget boot failure (detector: parent handshake timeout → Sentry event); tool-follow-up 400 (detector: assert no `LLM follow-up call failed` during a forced-tool synthetic run, and alert on that log line); guardrail judge fail-open to regex (C7); share-replay cache staleness and `.tmp` blob loss (detector: `img.onerror` beacon); Turnstile misconfiguration (detector: a 5-minute synthetic end-to-end run of the real form comparing preview-POST vs init-POST counts — exactly the C2 ratio).


=== Memo 2 ===

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
