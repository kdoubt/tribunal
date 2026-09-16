# Decision G09 - is a public live-preview demo funnel fit to put in front of prospects?

## Context (one paragraph, so the artifact makes sense)

TriageCRM is a pre-revenue HIPAA-oriented intake-chatbot + CRM for small
healthcare practices (dental, therapy, chiropractic, med-spa, physical
therapy, psychology). Its top-of-funnel is a **self-serve public demo** at
`/live-preview`: a prospect types their practice website URL, name and email;
the app screenshots their real site, detects their vertical, and overlays a
branded chat widget on that screenshot so they can talk to "their own" bot.
A CRM simulator panel fills in beside it as the conversation extracts a
contact and a lead, and the run can be shared as a replay link. The product
owner reported the day before the captures that this demo "appears to be broken". Live
captures from that day are an artifact below.

## Artifact(s)

Paths are relative to your current working directory (the artifact root).
Read anything in this tree; it is all in bounds, including files not listed
here. Nothing outside this tree is in bounds.

- `apps/web/src/pages/live-preview.astro` — the public demo page
- `apps/web/src/scripts/live-preview/` — its client controller (form,
  validation, prefetch, animation, simulator-bridge, replay-bridge, loading)
- `apps/web/src/components/crm-simulator/` — the CRM panel React component
- `apps/crm/src/app/api/public/live-preview/` — the public demo API
  (`route.ts`, `prefetch/`, `screenshot/[id]/`, `share/[token]/`,
  `quick-start/`) and its tests
- `apps/crm/src/lib/live-preview/` — demo-session, demo-share, screenshot
  service + signing, industry detection, page-text, trust gate, semaphore,
  circuit breaker, domain cache, storage, telemetry, and their tests
- `apps/crm/src/app/api/chat/widget/` — the chat backend the demo widget
  talks to (`init/route.ts`, `route.ts`, `llm-pipeline.ts`, `guardrails.ts`,
  `tool-executor.ts`, `session-lock.ts`, `session-loader.ts`,
  `persist-finalize.ts`, `demo-fallbacks.ts`, `helpers.ts`) and its tests
- `apps/chat/app/` — the widget SPA itself (TanStack Start): `client.tsx`,
  `router.tsx`, `routes/widget/$token.tsx`, `components/ChatWidget.tsx`,
  `components/PreChatForm.tsx`, `lib/api.ts`
- `packages/shared/src/flow-templates.ts` — the six vertical templates
- `packages/ai/src/extraction/` — industry/config/flow extraction
- `context/schema.prisma` — the full database schema
- `context/litellm-routing-SANITIZED.yaml` — the model routing actually
  deployed in front of this app (API keys removed; line numbers preserved)
- `context/production-captures-2026-09-12.md` — **C1–C10**, operator
  captures from the live production deployment, with method stated per
  capture. Cite these as `EXTERNAL` with this file as source.

Note on faithfulness: the source tree was copied from the running app host on
2026-09-12, so it is the deployed code, not a branch. Two known drifts are
stated in C7/C9 and you should treat them as facts, not infer others.

## Question under review

- **Q1.** Is this demo funnel fit to put in front of prospects as it stands?
  If not, what is the **minimum** set of changes that makes it fit — and what
  is explicitly *not* worth doing now?
- **Q2.** What best explains capture **C3** — the widget document and its two
  entry chunks served 200, the lazily-imported route chunk never requested,
  and `POST /app/api/chat/widget/init` never issued — and what change would
  make that class of failure either impossible or immediately visible?
- **Q3.** Which failures in this funnel are **silent** (the prospect sees a
  degraded or dead demo and nobody is told)? For each, name the cheapest
  detector that would have surfaced it within minutes.

## Decision criteria (owner-supplied)

Ranked. Where they conflict, higher wins.

1. **A prospect must never see a dead or obviously-wrong demo.** This is the
   entire top of funnel; a broken first impression is unrecoverable.
2. **Silent failure is worse than loud failure.** A defect that degrades the
   demo without telling anyone is treated as more severe than an equally
   large defect that throws.
3. **Build as if there were paying users** for anything hard to reverse
   (data model, auth boundary, contract shape) — but this does **not**
   license speculative features. Minimum scope for everything else.
4. **Cheap, checkable fixes beat elegant redesigns.** Prefer a change that an
   oracle can verify today.
5. **HIPAA posture is not optional**, but the demo handles synthetic patients
   only; do not inflate demo-only PHI exposure into a blocker unless real
   patient data can reach it.

Additive lenses — every seat addresses all of these in its own full read, and
none of them is anyone's exclusive slice: **correctness**, **operational
timing / integration completeness**, **observability**, **security/abuse of a
public unauthenticated endpoint**, and **prospect-facing polish**.

Tie-break pre-delegation: none. Unresolved decision-relevant disputes go to
the human.

## A. Operational readiness (mandatory — answer these or say "not applicable
and why")

- **A1.** For every dependency this funnel creates or waits on (screenshot
  render, demo session, demo share row, widget session, LLM call, tool call):
  what is the readiness predicate, poll interval, timeout, and retry policy?
  Flag anything async that is probed **once**.
- **A2.** Which writes in this flow are eventually consistent, and where does
  a reader assume otherwise? Include cache/negative-cache behaviour.
- **A3.** Where does the funnel confirm an externally-visible resource is
  live by asking the same party that just created it (self-confirmation), and
  what independent check should replace it?
- **A4.** Enumerate **every** store that must change for one demo run to be
  correct (DB tables, on-disk/blob screenshot storage, in-process caches,
  Redis/LLM cache, browser history/URL state, CDN cache). For each: who else
  writes it, and the symptom if it is skipped.
- **A5.** If a run aborts mid-way (browser closed, process restarted,
  semaphore lost), is the flow idempotent on resume, and what is orphaned?

## B. Precedent harvest (existing findings you should not re-derive)

Standing lessons from this estate, supplied as context; you may agree or
disagree with them, but do not present them as new discoveries:

- *"Verify the outcome, not the script's own logs"* — checking a producer's
  status output instead of the destination has repeatedly hidden real
  failures here, in both directions (a "FAILED" log while the work landed;
  a green build that never reached the browser).
- *"Ship the detector with the mechanism"* — in one earlier engineering review, 5 of 5
  defects in a single session failed **silently**; the standing rule is that
  a fix must come with a test that breaks the mechanism and asserts the
  detector fires. Absence of an alarm is never evidence of health.
- *Review brief gate* — the defect class that has cost this estate the most
  time is operational timing / eventual consistency / integration
  completeness, not security or concurrency. Hence section A.
- *2026-08-21, this project* — a separate Cloudflare Worker
  (`workers/chatbot-api`) was **demoted**; its production routes were
  disabled 2026-05-30 as a divergent second runtime. Live widget traffic goes
  through the CRM origin only. Do not propose routing demo traffic back
  through a second runtime without addressing why that was retired.
- *Also 2026-08-21* — a stale checkout of this project on another host
  carries an equivalent-looking fix that was deliberately never pushed. Code
  identity here is the app host's tree, which is what you have.

## C. Dependency fidelity (mandatory)

Where your claim's correctness depends on how a third-party component
behaves — LiteLLM's parameter handling, Anthropic's tool-calling
requirements, TanStack Start/Vite lazy route-chunk loading, Cloudflare
Turnstile, Playwright/Chromium, Prisma raw SQL identifier folding — you must
either cite the real behaviour (source line, changelog, doc, or one of the
C1–C10 captures) or label the claim `SPECULATIVE` and name the live probe
that would settle it. A claim resting on how a test's **fake** of that
dependency behaves is not grounded; say so explicitly if you find one.

## Constraints

- One product owner, no engineering team. Anything proposed must be
  implementable and verifiable by one person in hours, not weeks — or be
  explicitly labeled as a larger bet for the human to decide.
- Pre-revenue. No budget for paid monitoring SaaS; self-hosted and
  already-present tooling only (the estate already runs PostHog, Sentry is
  wired in this app, and there is a NATS spine and an existing dead-man
  watchdog pattern).
- The demo is **public and unauthenticated**. It performs an outbound
  screenshot fetch of a user-supplied URL and spends LLM tokens per visitor.
- Changes land on a single app host; no blue/green. Reversibility matters.
- Do not propose adding a second frontend runtime (see B).

## Output contract

Maximum 8 claims, each as:

```
CLAIM: <one sentence>
EVIDENCE: <a DECISIVE pointer — the smallest file:line, verbatim span, named
          symbol, capture ID (C1–C10), or oracle invocation that could settle
          the claim, not an argument; or ASSUMPTION; or SPECULATIVE (name the
          unreadable dependency); or EXTERNAL with source>
CONFIDENCE: <a 0-1 probability; calibrate, it is scored against oracle
          outcomes later>
FALSIFIER: <what concrete observation would prove this claim wrong>
```

Then **VERDICT INPUT** — one line per question (Q1, Q2, Q3).

Maximum **1200 words** total. Prose/markdown only. No praise, no preamble.
