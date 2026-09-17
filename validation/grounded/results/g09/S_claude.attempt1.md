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
