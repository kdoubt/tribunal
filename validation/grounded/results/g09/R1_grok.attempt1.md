I'll re-read the cited pointers against the frozen brief so the cross-exam attacks rest on the artifact, not on Round 0 memory.Cited files live under `artifact/`; I'll open the captures and the pointers the peer claims rest on.Peer A-section and Turnstile/route details still need a line-level check before I attack confidence.1. **ATTACK**

**Peer-1.** Token length is true (`demo-share.ts:20` vs `$token.tsx:8`) and the oracle already closed it — but it cannot be C3’s cause for a stronger reason the claim never cites: Zod runs inside `$token.tsx`, and C3 never requested `_token-*.js`. Search validation did not execute. 0.88 is a length fact, not a C3 ruling.

**Peer-2.** Class statement is right; **“structurally unobservable” at 0.75 is not.** C3 *is* the observation (document + two entry chunks, no `_token`, no init). Sentry lives in `__root.tsx:8`, which `routeTree.gen.ts:13` statically imports into the already-fetched ~98KB client. If that graph evaluated, Sentry would be up *before* the lazy `$token` import; if it never evaluated, nginx and a parent watchdog still see it. Unobservable is the *trigger*, not the class. `client.tsx:6-8` also overstates: `createRouter()` plus hydrate both have to run.

**Peer-3.** The env split and sync read are real (`live-preview.astro:98` vs `route.ts:133`; `form.ts:83-86`; `TURNSTILE_REQUIRED` ∉ `TRANSIENT_CODES`). Calling them a **live funnel-killer at 0.8** is not. C3’s `POST /live-preview` was **200** — Turnstile did not produce the reported dead demo. C10 is fail-closed and loud (400 + error step, retry hidden). Criterion 2 ranks that below silent widget death. Headless brunhild failures are the gate working. No evidence site key and secret currently disagree.

**Peer-4.** “No shipped observability” is false. `LIVE_PREVIEW_SCREENSHOT_SUCCESS` (`route.ts:406`) and `live_preview_widget_init` (`ChatWidget.tsx:78`) are shipped; the dead `telemetry.ts` ring buffer is a *second* unused path. 0.85 does not survive those two events. What is missing is an **alert** on their ratio (C2: 3 preview POSTs / 1 init).

**Peer-5.** Yaml shows two `triagecrm-chat` deployments and `latency-based-routing` (`:11-13`, `:25-27`, `:56`). That does **not** explain C7’s run-1/run-2 split. C7’s error is Anthropic-specific; run 2 clean is equally consistent with **no native tool_call**, so the follow-up path never ran (C6 run 2 chips). Latency routing *prefers* the faster deployment, which predicts stickiness, not a flip in minutes. `Available Model Group Fallbacks=None` is not routing. Section C: this is SPECULATIVE without LiteLLM logs. 0.62 is still an explanation, not a candidate.

**A-section.** A1 “demo session never re-read” is wrong: `init/route.ts:27` `getDemoSession`. A4 lists LiteLLM cache as a store that *must* change — skip does not break a run. Rest of A is mostly sound.

2. **CONCEDE**

C3 class = client graph never reached `/widget/$token` (C4 rules out URL/iframe, C5 the edge). Trigger SPECULATIVE. Eager-import/modulepreload + parent watchdog. `demoToken` length is not C3. Turnstile build/runtime split is a deploy footgun (loud). `getMetricsSummary` has no caller. Screenshot: 20s, `retries ?? 0` on POST (prefetch uses 1), probe-once. iframe `src` assignment is self-confirmation (`form.ts:151-152`). Share insert-then-update is two-step (`route.ts:325`). Resume not idempotent. Replay `img.src` has no `onerror` (`form.ts:349`) against `EXPIRY_DAYS = 30` and `.tmp` fallback (`constants.ts:3`).

3. **REVISE**

**Own-2** (0.78 → 0.84): C3 is the CSR shell + two entry modules loading while ChatWidget/init live in a lazy `/widget/$token` chunk the graph never resolved (not merely “parser never referenced”). Same falsifier.

**Own-3** (0.70, no shift): add C5 as ruling out the edge; trigger still SPECULATIVE.

**Own-7** (0.84 → 0.86): add (5) replay/share screenshot 404 with no `img.onerror` — cheapest detector: screenshot GET 404 after share 200, or an `error` listener → PostHog.

**Own-1,4,5,6,8:** no revision. Turnstile is loud/fail-closed, not min-fit; C7 intermittency stays unexplained.

4. **VERDICT INPUT**

**Q1:** NOT fit. Min: parent 5s zoom → error+event; eager-import/modulepreload `$token`; pass `tools` on follow-up; drop extra “ practice”; Own-7 detectors. Not now: second runtime, screenshot rewrite, paid APM, TanStack swap, demo-PHI, Turnstile redesign.

**Q2:** Graph never reached lazy `$token` (no chunk GET, no init); trigger unknown. Impossible via eager/modulepreload; visible via parent watchdog.

**Q3:** Dead widget (nginx 15s + parent timeout); C7 (`LLM follow-up call failed`); share-append (`[demo-share] Failed` / `messages:0`); PostHog screenshot without `live_preview_widget_init`; replay broken image (screenshot 404 / `onerror`).
