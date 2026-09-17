You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: ** C3 most likely represents client bootstrap/hydration failure before route execution, but its exact cause is unproven.
POINTER: apps/chat/app/client.tsx:6-8
OPENED TEXT:
const router = createRouter();

hydrateRoot(document, <StartClient router={router} />);

[2] CLAIM: ** C3 most likely represents client bootstrap/hydration failure before route execution, but its exact cause is unproven.
POINTER: ChatWidget.tsx:94-109
OPENED TEXT:
useEffect(() => {
    let cancelled = false;

    async function init() {
      try {
        // Demo mode: show animated pre-chat form IMMEDIATELY while init runs in parallel
        if (demoSessionToken) {
          // Show form right away with default styling — no await before render
          setShowPreChat(true);
          setLoading(false);

          // Fire full init in background with demo pre-chat data so the LLM skips name/email collection
          demoInitPromiseRef.current = initWidgetSession(chatbotToken, demoSessionToken, templateId, {
            preChatData: { firstName: 'Sarah', lastName: 'Johnson', email: '<email>' },
            demoToken,
          });

          // Update brand color/orgName once the init resolves (form is already visible)
          demoInitPromiseRef.current.then((data) => {

[3] CLAIM: ** Screenshot readiness has incomplete deadlines and a lease shorter than permitted retry work.
POINTER: prefetch/route.ts:19
OPENED TEXT:
acquireSemaphore,
  releaseSemaphore,
  getScreenshot,
  getMetadata,
  persistDemoMetadata,

[4] CLAIM: ** Screenshot readiness has incomplete deadlines and a lease shorter than permitted retry work.
POINTER: semaphore.ts:17-27
OPENED TEXT:
-- Clean expired holders
redis.call('ZREMRANGEBYSCORE', key, 0, now)

-- Check capacity
local count = redis.call('ZCARD', key)
if count >= max then
  return 0
end

-- Add holder with expiry as score
redis.call('ZADD', key, now + ttl_ms, holder)
redis.call('PEXPIRE', key, ttl_ms)
return 1
`;

[5] CLAIM: ** Session, share, widget, and tool readiness mostly relies on awaited calls without end-to-end deadlines or reconciliation.
POINTER: route.ts:292-328
OPENED TEXT:
// Create server-side demo session instead of HMAC-signed token
      const demoSessionId = await createDemoSession({
        contactId: contact.id,
        orgId: demoOrg.id,
        brandColor,
        orgName: siteName || businessName,
        chatbotToken,
        ...(bookingUrl ? { bookingUrl } : {}),
      });

      // Create shareable demo token + mint signed screenshot URL.
      //
      // TENANT-04 (Phase 26): the persisted DemoShare.screenshotUrl AND the
      // JSON response BOTH carry an HMAC-signed share-scoped token. The
      // share token is the OUTPUT of createDemoShare(), so we create the row
      // first with a placeholder and then update it with the signed URL.
      let demoToken: string | undefined;
      let signedScreenshotPath: string;
      try {
        demoToken = await createDemoShare({
          organizationId: demoOrg.id,
          contactId: contact.id,
          websiteUrl: url,
          screenshotUrl: '', // placeholder; updated to signed URL below
          brandColor,
          orgName: siteName || businessName,
          templateId: detectedIndustry || undefined,
        });
        const screenshotToken = mintScreenshotToken({
         

[6] CLAIM: ** Session, share, widget, and tool readiness mostly relies on awaited calls without end-to-end deadlines or reconciliation.
POINTER: init/route.ts:113-221
OPENED TEXT:
}

    const { url, businessName, email, phone } = parsed.data;
    const prefetchId = typeof rawPrefetchId === 'string' && /^[a-f0-9]{32}$/.test(rawPrefetchId)
      ? rawPrefetchId
      : null;

    // Trust controls: honeypot + prefetch timing
    const prefetchElapsedMs = prefetchStartedAt !== null ? Date.now() - prefetchStartedAt : null;
    const trustResult = evaluateTrust({
      honeypotFilled: !!honeypot,
      prefetchElapsedMs,
    });

    if (!trustResult.trusted) {
      // Silently reject suspicious requests with a generic success-like response
      console.log(`[trust:rejected] ip=${ip} reason=${trustResult.reason}`);
      return apiError('REJECTED', 'Unable to process request.', 400);
    }

    // Turnstile verification — enforce only when configured
    const turnstileConfigured = !!process.env.TURNSTILE_SECRET_KEY;
    if (turnstileConfigured && !turnstileToken) {
      return apiError('TURNSTILE_REQUIRED', 'Bot verification required', 400);
    }
    if (turnstileToken) {
      const turnstileValid = await verifyTurnstile(turnstileToken, ip);
      if (!turnstileValid) {
        return apiError('TURNSTILE_FAILED', 'Bot verification failed. Please try again.

[7] CLAIM: ** Session, share, widget, and tool readiness mostly relies on awaited calls without end-to-end deadlines or reconciliation.
POINTER: lib/api.ts:40-49
OPENED TEXT:
if (options?.demoToken) payload.demoToken = options.demoToken;
  const res = await fetch(`${API_BASE}/widget/init`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload),
  });
  if (!res.ok) {
    const data = await res.json().catch(() => ({}));
    throw new Error(data.error || 'Failed to initialize chat');
  }
  return res.json();
}

export async function sendWidgetMessage(

[8] CLAIM: ** Replay consistency and producer-side success checks can disagree with what prospects receive.
POINTER: route.ts:396-434
OPENED TEXT:
// LAUNCH-06: PostHog conversion funnel — screenshot_success
      // Fires after the screenshot is captured/stored, the demo session is
      // created, the DemoShare row exists, and the response is about to be
      // returned. Payload contains NO PHI: organizationId is a server UUID,
      // url_domain is hostname-only (never path/query), duration_ms is
      // millisecond elapsed from handler entry.
      try {
        const posthog = getPostHog();
        if (posthog) {
          posthog.capture({
            distinctId: demoSessionId,
            event: EVENTS.LIVE_PREVIEW_SCREENSHOT_SUCCESS,
            properties: {
              organizationId: demoOrg.id,
              url_domain: (() => { try { return new URL(url).hostname; } catch { return 'unknown'; } })(),
              duration_ms: Date.now() - requestStart,
            },
          });
        }
      } catch (posthogErr) {
        logger.error('[posthog] screenshot_success tracking failed', {
          request_id: requestId,
          org_id: demoOrg.id,
          route,
          outcome: 'error',
          latency_ms: Date.now() - requestStart,
          error_class: posthogErr instanceof Error ? posthogErr.n

[9] CLAIM: ** A run spans independently updated stores and is not idempotent across interruption.
POINTER: route.ts:243-383
OPENED TEXT:
// Find or create contact in demo org (transactional)
      const contact = await prisma.$transaction(async (tx) => {
        let existing = await tx.contact.findFirst({
          where: { organizationId: demoOrg.id, emailHash: blindIndex(email) },
        });

        if (existing) {
          if (existing.company !== businessName) {
            existing = await tx.contact.update({
              where: { id: existing.id },
              data: { company: businessName },
            });
          }
        } else {
          const nameParts = businessName.split(' ');
          existing = await tx.contact.create({
            data: {
              organizationId: demoOrg.id,
              firstName: nameParts[0] || 'Demo',
              lastName: nameParts.slice(1).join(' ') || 'User',
              email,
              emailHash: blindIndex(email),
              phone: phone || null,
              company: businessName,
            },
          });
        }

        if (demoOrg.tags[0]) {
          await tx.tagAssignment.upsert({
            where: {
              tagId_contactId: {
                tagId: demoOrg.tags[0].id,
                contactId: existing.id,
              },


[10] CLAIM: ** A run spans independently updated stores and is not idempotent across interruption.
POINTER: init/route.ts:110-221
OPENED TEXT:
const parsed = livePreviewRequestSchema.safeParse(body);
    if (!parsed.success) {
      return apiError('INVALID_INPUT', 'Invalid input', 400, parsed.error.flatten().fieldErrors as Record<string, unknown>);
    }

    const { url, businessName, email, phone } = parsed.data;
    const prefetchId = typeof rawPrefetchId === 'string' && /^[a-f0-9]{32}$/.test(rawPrefetchId)
      ? rawPrefetchId
      : null;

    // Trust controls: honeypot + prefetch timing
    const prefetchElapsedMs = prefetchStartedAt !== null ? Date.now() - prefetchStartedAt : null;
    const trustResult = evaluateTrust({
      honeypotFilled: !!honeypot,
      prefetchElapsedMs,
    });

    if (!trustResult.trusted) {
      // Silently reject suspicious requests with a generic success-like response
      console.log(`[trust:rejected] ip=${ip} reason=${trustResult.reason}`);
      return apiError('REJECTED', 'Unable to process request.', 400);
    }

    // Turnstile verification — enforce only when configured
    const turnstileConfigured = !!process.env.TURNSTILE_SECRET_KEY;
    if (turnstileConfigured && !turnstileToken) {
      return apiError('TURNSTILE_REQUIRED', 'Bot verification required', 400);
    }
 

[11] CLAIM: ** A run spans independently updated stores and is not idempotent across interruption.
POINTER: form.ts:124-128
OPENED TEXT:
// Reflect the demo token in the browser URL for easy sharing
    if (currentDemoToken) {
      const shareUrl = new URL(window.location.href);
      shareUrl.searchParams.set('d', currentDemoToken);
      window.history.replaceState({}, '', shareUrl.toString());
    }

    (document.getElementById('url-bar') as HTMLElement).textContent = data.url;

[12] CLAIM: ** A run spans independently updated stores and is not idempotent across interruption.
POINTER: session-lock.ts:4
OPENED TEXT:
const LOCK_TTL_S = 90;

export async function acquireSessionLock(
  sessionId: string,

[13] CLAIM: ** The post-tool completion contract explains C7 independently of C3.
POINTER: llm-pipeline.ts:180-199
OPENED TEXT:
try {
        const followUpMessages = [
          ...messages,
          { role: 'assistant' as const, content: assistantMessage.content || '', tool_calls: assistantMessage.tool_calls },
          ...toolResults,
        ] as Parameters<typeof client.chat.completions.create>[0]['messages'];

        // Phase 29 DEPLOY-03: route through instrumentedCompletion for histogram.
        const followUp = await withAbortableTimeout(
          (signal) =>
            instrumentedCompletion(
              { model: MODELS.CHAT, messages: followUpMessages, max_tokens: 1024 },
              { signal },
            ),
          30_000,
          'LLM follow-up',
        );
        reply = followUp.choices[0]?.message?.content || '';
      } catch (followUpErr) {
        console.error('LLM follow-up call failed:', followUpErr);
        reply = buildToolFallbackReply(assistantMessage.tool_calls || [], toolResults);
      }

      if (leadCreated) {

[14] CLAIM: ** The post-tool completion contract explains C7 independently of C3.
POINTER: __tests__/llm-pipeline-tool-drop.test.ts:23-28
OPENED TEXT:
return {
    getLLMClient: vi.fn().mockReturnValue({
      chat: { completions: { create: (...args: unknown[]) => llmCreateMock(...args) } },
    }),
    instrumentedCompletion: (...args: unknown[]) => llmCreateMock(...args),
    MODELS: { CHAT: 'fake-model' },
    qualifierTools: [],
    qualifierToolSchemas: tools.qualifierToolSchemas,
    retrieveContext: vi.fn().mockResolvedValue(''),
    getPromRegistry: vi.fn(() => ({})),

[15] CLAIM: ** Public abuse and replay boundaries need targeted gates, not a speculative security redesign.
POINTER: route.ts:42-76
OPENED TEXT:
export async function POST(request: NextRequest) {
  // LAUNCH-06: capture handler-entry timestamp so screenshot_success can
  // emit a duration_ms metric. Declared before ip so the helper closure
  // below can reference it without temporal-dead-zone issues.
  const requestStart = Date.now();
  const requestId = request.headers.get('x-request-id') ?? crypto.randomUUID();
  const route = 'public.live-preview';
  // Phase 27 DELIVERY-03: fail-closed posture matching chat widget.
  // ip === 'unknown' returns 400 IP_REQUIRED; Redis throw returns 503
  // SERVICE_UNAVAILABLE. No fail-OPEN swallow remains.
  const ip = getRateLimitIp(request);
  if (ip === 'unknown') return apiError('IP_REQUIRED', 'Unable to identify client', 400);
  // LAUNCH-06: PostHog conversion funnel — screenshot_fail helper.
  // Called immediately before every apiError(...) exit that represents a
  // screenshot-pipeline failure (not rate-limit/IP/validation early exits —
  // those are pre-screenshot abuse controls). Best-effort: capture failures
  // never abort the primary apiError response. distinctId is the client IP
  // because demoSession isn't created yet at most failure points.
  const captureScreens

[16] CLAIM: ** Several degraded outcomes have no demonstrated operator notification despite existing logs and metrics.
POINTER: animation.ts:58-62
OPENED TEXT:
// Fallback: if the iframe never sends the message (e.g. load failure), zoom after 5s
  widgetZoomTimer = setTimeout(() => {
    triggerZoom();
    widgetZoomTimer = null;
  }, 5000);
}

export function resetWidgetAnimation(): void {

[17] CLAIM: ** Several degraded outcomes have no demonstrated operator notification despite existing logs and metrics.
POINTER: tool-executor.ts:107-123
OPENED TEXT:
const service = typeof args.service === 'string' ? args.service : String(args.service ?? '');
      let serviceInfo = `Yes, we can help with ${service}. If the patient wants to book, proceed to collect any missing required fields then create the lead.`;
      try {
        const context = await retrieveContext(session.organizationId, service, 1);
        if (context) {
          serviceInfo = `Service info: ${context}. If the patient wants to book, proceed to collect any missing required fields then create the lead.`;
        } else {
          const fallback = getDemoServiceFallback(service);
          if (fallback) serviceInfo = fallback;
        }
      } catch {
        const fallback = getDemoServiceFallback(service);
        if (fallback) serviceInfo = fallback;
      }
      toolResults.push({
        role: 'tool',
        tool_call_id: toolCall.id,
        content: JSON.stringify({ available: true, info: serviceInfo }),
      });
    }
  }

[18] CLAIM: ** Several degraded outcomes have no demonstrated operator notification despite existing logs and metrics.
POINTER: ChatWidget.tsx:224-235
OPENED TEXT:
// Forward CRM data to parent window for the live CRM simulator
      if (data.crmData) {
        window.parent.postMessage(
          { type: 'triagecrm:chat-message', payload: data.crmData },
          '*',
        );
      }
      if (data.done) {
        window.parent.postMessage(
          { type: 'triagecrm:chat-done', payload: data.crmData || {} },
          '*',
        );
      }
    } catch (err) {
      // LAUNCH-03: resolve the structured error code (preserved by

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }