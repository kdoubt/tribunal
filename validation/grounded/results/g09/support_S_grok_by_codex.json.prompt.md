You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The funnel is not fit for prospects: a generated preview can be shown as live with a non-interactive widget, plus doubled welcome copy and a silent post-tool LLM failure.
POINTER: live-preview.astro:124
OPENED TEXT:
<div class="text-center">
          <h2 class="text-2xl font-bold text-gray-900">Your AI chatbot is live!</h2>
          <p class="mt-2 text-gray-600">Chat with the AI below — it's real and fully functional.</p>
        </div>

[2] CLAIM: The funnel is not fit for prospects: a generated preview can be shown as live with a non-interactive widget, plus doubled welcome copy and a silent post-tool LLM failure.
POINTER: form.ts:149-152
OPENED TEXT:
showStep('preview');
    window.scrollTo({ top: 0, behavior: 'instant' as ScrollBehavior });
    (document.getElementById('preview-widget') as HTMLIFrameElement).src = widgetSrc;
    triggerWidgetAnimation();
  } catch {
    stopLoadingMessages();
    if (retryTimer) {

[3] CLAIM: C3 is the CSR widget shell (1293 B) plus two entry modules loading, while ChatWidget/`POST .../init` live in a lazy `/widget/$token` chunk the HTML parser never referenced, so that `import()` never ran.
POINTER: token.tsx:3
OPENED TEXT:
import { z } from 'zod';
import { ChatWidget } from '~/components/ChatWidget';

const widgetSearchSchema = z.object({
  demoSession: z.string().max(500).optional(),

[4] CLAIM: C3 is the CSR widget shell (1293 B) plus two entry modules loading, while ChatWidget/`POST .../init` live in a lazy `/widget/$token` chunk the HTML parser never referenced, so that `import()` never ran.
POINTER: ChatWidget.tsx:106
OPENED TEXT:
// Fire full init in background with demo pre-chat data so the LLM skips name/email collection
          demoInitPromiseRef.current = initWidgetSession(chatbotToken, demoSessionToken, templateId, {
            preChatData: { firstName: 'Sarah', lastName: 'Johnson', email: '<email>' },
            demoToken,
          });

[5] CLAIM: The C3 class becomes visible with a parent watchdog, and the “chunk never requested” signature becomes impossible if `/widget/$token` is eager in the already-fetched ~98KB client bundle (or modulepreloaded).
POINTER: animation.ts:58-62
OPENED TEXT:
// Fallback: if the iframe never sends the message (e.g. load failure), zoom after 5s
  widgetZoomTimer = setTimeout(() => {
    triggerZoom();
    widgetZoomTimer = null;
  }, 5000);
}

export function resetWidgetAnimation(): void {

[6] CLAIM: The C3 class becomes visible with a parent watchdog, and the “chunk never requested” signature becomes impossible if `/widget/$token` is eager in the already-fetched ~98KB client bundle (or modulepreloaded).
POINTER: PreChatForm.tsx:69-70
OPENED TEXT:
// Notify parent frame (live-preview page) to start the zoom animation
    const notifyTimer = setTimeout(() => {
      try { window.parent.postMessage({ type: 'triagecrm:prechat-complete' }, '*'); } catch { /* iframe sandbox */ }
    }, cursor);
    animationRef.current.push(notifyTimer);

[7] CLAIM: Demo welcome always appends “ practice” to the template name, so `psychology-practice` → “psychology practice practice.”
POINTER: init/route.ts:261
OPENED TEXT:
firstName: nameParts[0] || 'Demo',
              lastName: nameParts.slice(1).join(' ') || 'User',
              email,
              emailHash: blindIndex(email),
              phone: phone || null,

[8] CLAIM: Demo welcome always appends “ practice” to the template name, so `psychology-practice` → “psychology practice practice.”
POINTER: flow-templates.ts:543-544
OPENED TEXT:
{
    id: 'psychology-practice',
    name: 'Psychology Practice',
    industry: 'Psychology',
    description: 'Intent routing, state eligibility, telehealth/in-person, self-pay disclosure, and Calendly booking.',
    icon: 'Brain',

[9] CLAIM: After a native tool call, the follow-up completion omits `tools`, which Anthropic/LiteLLM 400s; the widget still 200s a canned fallback — tests fake this call and cannot catch it.
POINTER: llm-pipeline.ts:189-199
OPENED TEXT:
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

[10] CLAIM: After a native tool call, the follow-up completion omits `tools`, which Anthropic/LiteLLM 400s; the widget still 200s a canned fallback — tests fake this call and cannot catch it.
POINTER: litellm-routing-SANITIZED.yaml:43
OPENED TEXT:
litellm_settings:
  drop_params: true
  cache: true
  # HIPAA-mandatory: metadata-only telemetry to PostHog, no prompts/responses
  success_callback: ["posthog"]

[11] CLAIM: Silent prospect-facing failures and cheapest detectors: (1) widget GET without `_token` or init — nginx dead-man 15s + parent timeout event; (2) C7 follow-up — grep `LLM follow-up call failed`; (3) share JSON append — grep `[demo-share] Failed to record message` or share GET `messages: 0` after a 200 chat; (4) PostHog `screenshot_success` without `live_preview_widget_init`.
POINTER: persist-finalize.ts:119-121
OPENED TEXT:
}
    } catch (shareErr) {
      console.error('[demo-share] Failed to record message:', shareErr);
    }
  }

  return { crmData };

[12] CLAIM: Silent prospect-facing failures and cheapest detectors: (1) widget GET without `_token` or init — nginx dead-man 15s + parent timeout event; (2) C7 follow-up — grep `LLM follow-up call failed`; (3) share JSON append — grep `[demo-share] Failed to record message` or share GET `messages: 0` after a 200 chat; (4) PostHog `screenshot_success` without `live_preview_widget_init`.
POINTER: ChatWidget.tsx:78
OPENED TEXT:
// (no email, phone, firstName, sessionId, full token).
    capturePostHog('live_preview_widget_init', {
      org_token_prefix: chatbotToken.slice(0, 8),
    });

[13] CLAIM: Silent prospect-facing failures and cheapest detectors: (1) widget GET without `_token` or init — nginx dead-man 15s + parent timeout event; (2) C7 follow-up — grep `LLM follow-up call failed`; (3) share JSON append — grep `[demo-share] Failed to record message` or share GET `messages: 0` after a 200 chat; (4) PostHog `screenshot_success` without `live_preview_widget_init`.
POINTER: live-preview/route.ts:406
OPENED TEXT:
posthog.capture({
            distinctId: demoSessionId,
            event: EVENTS.LIVE_PREVIEW_SCREENSHOT_SUCCESS,
            properties: {
              organizationId: demoOrg.id,

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }