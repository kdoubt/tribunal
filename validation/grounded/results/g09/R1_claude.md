## 1. ATTACK

**Peer-1** (trigger unknown; C4(b) rules out blanket sandbox blocking). Directionally right, but two defects. (a) 0.70 is miscalibrated *downward*: C3 + C5 + C4 make "the trigger is not determinable from this tree" nearly certain — a claim of ignorance grounded in three captures deserves ≥0.9. (b) The falsifier is half-dead on arrival: "Owner DevTools showing `_token` requested" contradicts C3's nginx log, which is the same wire. The live half — a Sentry hydration exception — presumes a DSN: `sentry-init.ts:26-28` returns early unless `VITE_SENTRY_DSN_CHAT` was set **at build time**. If it was unset, an empty Sentry for 17:59–18:05 is not evidence of no throw (precedent B: absence of an alarm is never evidence of health). State that conditional or the probe misleads.

**Peer-2** (four silent failures + detectors). The failure inventory is right and the pointers resolve. The *detectors* are under-argued: items (2) and (3) are `grep`, which is an investigation, not a detector — nothing fires within minutes unless it is scheduled and routed. Item (4) is worse: C2 shows **3 preview POSTs in 15 days**. A PostHog funnel alarm has no base rate at that volume; it will never trip meaningfully. What survives at this traffic is a *per-session* server-side dead-man — on `createDemoSession`, arm a 90 s check that a `chat/widget/init` bearing that `demoSession` arrived — using the estate's existing watchdog pattern. 0.84 is fine for the enumeration, too high for "cheapest detector."

**Peer-3** (watchdog + eager import). The watchdog half is right and the emitter pointer (`PreChatForm.tsx:69-70`) is more precise than mine. The "impossible" half is overclaimed at 0.86. C3 shows the 98 KB entry chunk was *fetched* 200; it does not show it *executed*. If the entry graph never ran, or hydration threw, eager-importing `/widget/$token` removes the signature and leaves the failure — no `_token` GET to be missing, still no init POST. Eager import only cures a cause specific to the dynamic `import()` (chunk fetch blocked, CSP, extension). Second defect: a 5 s threshold keyed on *absence of* `prechat-complete` will false-positive. The legit message is ~1475 ms after `PreChatForm` mount (`PreChatForm.tsx:19-25,56-71`), but mount is gated on the init fetch resolving; a slow init eats the 5 s and the parent would report a healthy run as dead. The watchdog must key on a distinct `widget-ready` emitted at init-success, not on the animation's completion message.

**Peer-4** (do-not-do list). Agreed except one item. "Abuse controls already fail closed (Turnstile/trust 400s are loud)" conflates loud-to-the-log with loud-to-the-prospect. C10: hosts that cannot reach `brunhild.challenges.cloudflare.com` "were unable to complete the form at all," and `TURNSTILE_REQUIRED` is absent from `TRANSIENT_CODES` (`form.ts:8`), so the retry button is hidden. That is criterion-1 damage, in-scope, and fixable in hours. Also don't blanket-exclude storage: `demo-share.ts:7` `EXPIRY_DAYS=30` against a `.tmp` blob path plus `img.src` with no `onerror` (`form.ts:349`) is a one-line fix, not the "screenshot rewrite" you're excluding.

## 2. CONCEDE

- `PreChatForm.tsx:69-70` is the sole emitter of `triagecrm:prechat-complete` — supersedes my `ChatWidget.tsx:172` pointer.
- C4(b) does rule out "the iframe context always blocks it."
- Sentry *does* initialise in the entry graph (`__root.tsx:3,8`), so my "structurally unobservable" was too strong.
- Swallowed share-append (`persist-finalize.ts:119-121`) belongs in the silent-failure list.
- Peer-4's exclusions minus Turnstile/storage stand.

## 3. REVISE

- **Own-1**: evidence → `animation.ts:58-62` vs `PreChatForm.tsx:69-71`; add that the 5 s budget also spans the init round-trip, so the fallback can fire on a *healthy* slow run. 0.92 → 0.93.
- **Own-2**: drop "structurally unobservable"; new text: *a hydration throw would reach Sentry only if `VITE_SENTRY_DSN_CHAT` was set at build (`sentry-init.ts:26-28`); otherwise silence is uninformative.* 0.75 → 0.60.
- Others: no revision.

## 4. VERDICT INPUT

- **Q1.** Not fit. Minimum: `widget-ready` postMessage + parent error state (no blind zoom), server-side 90 s init dead-man, add `tools` to `llm-pipeline.ts:190`, fix the doubled "practice", fix the Turnstile render/await/retry-code trio. Not now: second runtime, screenshot/semaphore/breaker rewrite, paid APM, demo-PHI work.
- **Q2.** The route chunk was never requested because the client graph never reached route resolution; trigger undetermined. Make it *visible* (parent watchdog + server dead-man); eager import narrows but does not eliminate the class.
- **Q3.** Silent: no-init widget, C7 follow-up swallow, share-append swallow, expired-screenshot broken `img`. Detectors must be scheduled and per-session, not grep or funnel-rate alarms — traffic is ~3 events/15 days.
