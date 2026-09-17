# g09 oracles (orchestrator, mechanical: pointers opened; `verified` only where the cited text settles the claim by inspection)

grounding.py: S_claude 18/18 pointers open; S_grok 13/14 open by tool, the 14th (`route.test.ts:326`) is a same-basename
mismatch (six route.test.ts files) - opened by hand at `apps/crm/src/app/api/chat/widget/__tests__/route.test.ts:326-329`
and it IS the mocked follow-up (`mockCreate.mockResolvedValueOnce({... tool_calls: null })`). So 14/14.

| pointer | opened text (paraphrase of the literal lines) | settles |
|---|---|---|
| animation.ts:58-62 | `// Fallback: if the iframe never sends the message (e.g. load failure), zoom after 5s` → `setTimeout(triggerZoom, 5000)` | A1/B4 mechanism: unconditional zoom, no error path - VERIFIED |
| animation.ts:49-55 | message listener for `triagecrm:prechat-complete` → triggerZoom | same |
| ChatWidget.tsx:172 | `window.parent.postMessage({type:'triagecrm:chat-init'...})` inside `handlePreChatSubmit` | A1: widget's parent messages come only after boot - VERIFIED |
| PreChatForm.tsx:69-70 | `postMessage({ type: 'triagecrm:prechat-complete' })` on a timer inside the demo animation | B4: the only emitter of that message, emitted only after the route component mounted - VERIFIED |
| __root.tsx:3,8 | `import { initSentry }` ... `initSentry()` at module scope | A2: Sentry initialised inside the widget module graph - VERIFIED |
| client.tsx:6-8 | `createRouter(); hydrateRoot(document, <StartClient router={router}/>)` | A2: route resolution is a consequence of client execution - VERIFIED |
| $token.tsx:3 / ChatWidget.tsx:106 | `import { ChatWidget }` from the route file; `initWidgetSession(...)` fired from ChatWidget | B2: init is issued only from the route component - VERIFIED |
| $token.tsx:8 / demo-share.ts:20 | `demoToken: z.string().max(20)`; `randomBytes(15).toString('base64url').slice(0, 20)` | A3: token length within bound - VERIFIED |
| llm-pipeline.ts:190 vs :100-102 | follow-up `{ model, messages, max_tokens }` (no tools) vs first call `tools: qualifierTools, tool_choice: 'auto'` | A4/B6 - VERIFIED |
| llm-pipeline.ts:197-200 | `catch (followUpErr) { console.error('LLM follow-up call failed:'...); reply = buildToolFallbackReply(...) }` | A4/B6 swallow-to-fallback - VERIFIED |
| litellm-routing-SANITIZED.yaml:42-52 (file lines 72-82) | `drop_params: true`, no `modify_params` | A4/B6 - VERIFIED |
| litellm-routing-SANITIZED.yaml:11-13,25-27,56 (file lines 21-23,45-47,85) | two `triagecrm-chat` deployments (anthropic haiku 4.5; cf llama-3.1-70b); `routing_strategy: latency-based-routing` | A5 premise (two deployments, latency routing) VERIFIED; the inference that this explains C7's intermittency is PANELIST-CLAIM |
| route.test.ts:326-329 | mock follow-up returns content with `tool_calls: null` | B6 "tests fake this call" - VERIFIED |
| live-preview.astro:98 / route.ts:133 | `import.meta.env.PUBLIC_TURNSTILE_SITE_KEY && (<div class="cf-turnstile">)` vs `!!process.env.TURNSTILE_SECRET_KEY` | A6 premise (build-time render gate vs runtime enforcement) - VERIFIED |
| form.ts:83-86 / :8 / :31-35 | reads `[name="cf-turnstile-response"]` at submit, no await; `TRANSIENT_CODES` lacks TURNSTILE_REQUIRED; non-retryable hides retry button | A6 - VERIFIED |
| C10 | curl without token → 400 TURNSTILE_REQUIRED; headless browsers unable to reach challenges.cloudflare.com could not complete the form | A6 corroboration and B8's "loud 400" - both literally true |
| init/route.ts:261 / flow-templates.ts:543-544 / C8 | `` `...you're a ${templateName.toLowerCase()} practice...` ``; `name: 'Psychology Practice'`; rendered "psychology practice practice" | A7/B5 - VERIFIED for psychology; "all six verticals" (A7) not settled by these lines |
| telemetry.ts:15-17, 54-58 | in-process arrays `metrics`, `latencies`, MAX 1000; `logEvent` = console.log JSON | A8 premise - VERIFIED; `getMetricsSummary` is re-exported (index.ts:34) with no other caller found by grep |
| persist-finalize.ts:119-121 | `catch (shareErr) { console.error('[demo-share] Failed to record message:' ...) }` | B7(3) - VERIFIED |
| ChatWidget.tsx:78 / live-preview/route.ts:406 | `capturePostHog('live_preview_widget_init')`; `EVENTS.LIVE_PREVIEW_SCREENSHOT_SUCCESS` | B7(4): both PostHog events exist - VERIFIED (bears on A8's "nothing correlates") |
| live-preview.astro:124 / form.ts:149-152 | "Your AI chatbot is live!" heading; `showStep('preview')` then `iframe.src = widgetSrc` | B1 - VERIFIED |
| C2, C3, C4, C5, C7 | as captured (EXTERNAL) | quoted correctly by both seats |

Not settled by inspection (stay PANELIST-CLAIM / SPECULATIVE): A2's and B3's "why the import never ran"; A5's routing explanation of C7 intermittency;
whether a modulepreload of `_token-*.js` exists in the served HTML (no built HTML in the tree; C3 shows it was not requested).
