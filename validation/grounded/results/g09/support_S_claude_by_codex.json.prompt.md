You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: ** The funnel is not fit as-is: a widget that fails to boot is rendered as a *successful* demo, because the parent page's only post-src behaviour is an unconditional 5-second zoom, and nothing in the widget ever signals "ready".
POINTER: apps/web/src/scripts/live-preview/animation.ts:58-62
OPENED TEXT:
// Fallback: if the iframe never sends the message (e.g. load failure), zoom after 5s
  widgetZoomTimer = setTimeout(() => {
    triggerZoom();
    widgetZoomTimer = null;
  }, 5000);
}

export function resetWidgetAnimation(): void {

[2] CLAIM: ** The funnel is not fit as-is: a widget that fails to boot is rendered as a *successful* demo, because the parent page's only post-src behaviour is an unconditional 5-second zoom, and nothing in the widget ever signals "ready".
POINTER: apps/chat/app/components/ChatWidget.tsx:172
OPENED TEXT:
// Notify parent window of initial contact data for CRM simulator
      window.parent.postMessage(
        {
          type: 'triagecrm:chat-init',
          payload: { firstName: data.firstName, lastName: data.lastName, email: data.email },

[3] CLAIM: ** C3's class is "the widget's client module graph never executed to route resolution", and it is structurally unobservable because the route chunk is requested *only* as a consequence of that execution and Sentry is initialised inside the very graph that failed.
POINTER: apps/chat/app/routes/__root.tsx:3
OPENED TEXT:
import appCss from '~/styles/global.css?url';
import { initSentry } from '~/lib/sentry-init';

// Phase 29 DEPLOY-03: Sentry init at module-load time (browser-only via
// typeof window guard inside initSentry()). No-op when VITE_SENTRY_DSN_CHAT

[4] CLAIM: ** C3's class is "the widget's client module graph never executed to route resolution", and it is structurally unobservable because the route chunk is requested *only* as a consequence of that execution and Sentry is initialised inside the very graph that failed.
POINTER: apps/chat/app/client.tsx:6-8
OPENED TEXT:
const router = createRouter();

hydrateRoot(document, <StartClient router={router} />);

[5] CLAIM: ** Search-param validation is *not* the cause: `demoToken` is exactly 20 chars, within the schema bound.
POINTER: demo-share.ts:20
OPENED TEXT:
export async function createDemoShare(data: DemoShareData): Promise<string> {
  const token = randomBytes(15).toString('base64url').slice(0, 20);
  const expiresAt = new Date(Date.now() + EXPIRY_DAYS * 24 * 60 * 60 * 1000);

  await prisma.demoShare.create({

[6] CLAIM: ** Search-param validation is *not* the cause: `demoToken` is exactly 20 chars, within the schema bound.
POINTER: token.tsx:8
OPENED TEXT:
templateId: z.string().max(50).optional(),
  demoToken: z.string().max(20).optional(),
});

export const Route = createFileRoute('/widget/$token')({

[7] CLAIM: ** The post-tool follow-up LLM call omits `tools`, which LiteLLM rejects for Anthropic deployments, so the demo's money moment (tool call → contact/lead) degrades to canned copy with only a `console.error`.
POINTER: llm-pipeline.ts:190
OPENED TEXT:
instrumentedCompletion(
              { model: MODELS.CHAT, messages: followUpMessages, max_tokens: 1024 },
              { signal },
            ),
          30_000,

[8] CLAIM: ** The post-tool follow-up LLM call omits `tools`, which LiteLLM rejects for Anthropic deployments, so the demo's money moment (tool call → contact/lead) degrades to canned copy with only a `console.error`.
POINTER: context/litellm-routing-SANITIZED.yaml:42-52
OPENED TEXT:
# lines 72-82, verbatim
litellm_settings:
  drop_params: true
  cache: true
  # HIPAA-mandatory: metadata-only telemetry to PostHog, no prompts/responses
  success_callback: ["posthog"]
  failure_callback: ["posthog"]
  turn_off_message_logging: True
  cache_params:
    type: "redis"
    host: "redis"
    port: 6379

# lines 84-88, verbatim
router_settings:

[9] CLAIM: ** The post-tool follow-up LLM call omits `tools`, which LiteLLM rejects for Anthropic deployments, so the demo's money moment (tool call → contact/lead) degrades to canned copy with only a `console.error`.
POINTER: llm-pipeline.ts:197-200
OPENED TEXT:
reply = followUp.choices[0]?.message?.content || '';
      } catch (followUpErr) {
        console.error('LLM follow-up call failed:', followUpErr);
        reply = buildToolFallbackReply(assistantMessage.tool_calls || [], toolResults);
      }

      if (leadCreated) {
        done = true;

[10] CLAIM: ** The intermittency of C7 (run 1 fails, run 2 clean) is explained by `triagecrm-chat` being a two-deployment group under latency-based routing, so demo replies come non-deterministically from Anthropic Haiku 4.5 or Cloudflare llama-3.1-70b.
POINTER: litellm-routing-SANITIZED.yaml:11-13
OPENED TEXT:
# line 21
  - model_name: "triagecrm-chat"
    litellm_params:
      model: "anthropic/claude-haiku-4-5-20251001"   # line 23
  # line 29
  - model_name: "triagecrm-assistant"
    litellm_params:

[11] CLAIM: ** Turnstile is a live funnel-killer with two independent defects: the gate is enabled by a *server* env var but rendered from a *build-time* client var, and the client reads the token synchronously without waiting for the challenge.
POINTER: live-preview.astro:98
OPENED TEXT:
<!-- Turnstile widget (only rendered when sitekey is configured) -->
          {import.meta.env.PUBLIC_TURNSTILE_SITE_KEY && (
            <div class="cf-turnstile" data-sitekey={import.meta.env.PUBLIC_TURNSTILE_SITE_KEY}></div>
          )}

[12] CLAIM: ** Turnstile is a live funnel-killer with two independent defects: the gate is enabled by a *server* env var but rendered from a *build-time* client var, and the client reads the token synchronously without waiting for the challenge.
POINTER: route.ts:133
OPENED TEXT:
// Turnstile verification — enforce only when configured
    const turnstileConfigured = !!process.env.TURNSTILE_SECRET_KEY;
    if (turnstileConfigured && !turnstileToken) {
      return apiError('TURNSTILE_REQUIRED', 'Bot verification required', 400);
    }

[13] CLAIM: ** Turnstile is a live funnel-killer with two independent defects: the gate is enabled by a *server* env var but rendered from a *build-time* client var, and the client reads the token synchronously without waiting for the challenge.
POINTER: form.ts:83-86
OPENED TEXT:
// Include Turnstile token if available
    const turnstileInput = document.querySelector<HTMLInputElement>('[name="cf-turnstile-response"]');
    if (turnstileInput?.value) {
      payload['cf-turnstile-response'] = turnstileInput.value;
    }

    const res = await fetch('/app/api/public/live-preview', {
      method: 'POST',

[14] CLAIM: ** Turnstile is a live funnel-killer with two independent defects: the gate is enabled by a *server* env var but rendered from a *build-time* client var, and the client reads the token synchronously without waiting for the challenge.
POINTER: form.ts:8
OPENED TEXT:
const TRANSIENT_CODES = new Set(['SCREENSHOT_TIMEOUT', 'SCREENSHOT_UNAVAILABLE', 'BUSY', 'SCREENSHOT_BUSY']);

let retryCount = 0;
let retryTimer: ReturnType<typeof setTimeout> | null = null;

[15] CLAIM: ** Turnstile is a live funnel-killer with two independent defects: the gate is enabled by a *server* env var but rendered from a *build-time* client var, and the client reads the token synchronously without waiting for the challenge.
POINTER: form.ts:31-35
OPENED TEXT:
const retryBtn = document.getElementById('retry-btn')!;
  if (retryable) {
    retryBtn.classList.remove('hidden');
  } else {
    retryBtn.classList.add('hidden');
  }
  showStep('error');
}

[16] CLAIM: ** The demo's first sentence is grammatically broken for all six verticals.
POINTER: init/route.ts:261
OPENED TEXT:
firstName: nameParts[0] || 'Demo',
              lastName: nameParts.slice(1).join(' ') || 'User',
              email,
              emailHash: blindIndex(email),
              phone: phone || null,

[17] CLAIM: ** The demo's first sentence is grammatically broken for all six verticals.
POINTER: flow-templates.ts:544
OPENED TEXT:
id: 'psychology-practice',
    name: 'Psychology Practice',
    industry: 'Psychology',
    description: 'Intent routing, state eligibility, telehealth/in-person, self-pay disclosure, and Calendly booking.',
    icon: 'Brain',

[18] CLAIM: ** There is no shipped observability for this funnel: the live-preview telemetry module is an in-process ring buffer with no exporter, so nothing correlates "preview 200" with "widget init".
POINTER: lib/live-preview/telemetry.ts:15-17
OPENED TEXT:
const metrics: Metric[] = [];
const latencies: LatencyRecord[] = [];
const MAX_BUFFER_SIZE = 1000;

/**
 * Record a counter or gauge metric.

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }