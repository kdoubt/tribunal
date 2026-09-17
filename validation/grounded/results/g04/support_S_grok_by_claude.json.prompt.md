You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The landing’s primary voice is a human uploader unblocking Claude Code, not a terminal agent that captures pages itself.
POINTER: index.tsx:42-47
OPENED TEXT:
<h1>Your AI agent<br />can finally see.</h1>
          <p className="hero-explainer">
            You can't paste screenshots into Claude Code. WhipNode fixes that.
          </p>
          <div className="hero-cta">
            <a href="/new" className="btn btn-primary">Upload something</a>
            <a href="/pricing" className="btn btn-secondary">See how to scale</a>
          </div>
        </div>

[2] CLAIM: The landing’s primary voice is a human uploader unblocking Claude Code, not a terminal agent that captures pages itself.
POINTER: index.tsx:78
OPENED TEXT:
<h2>How it works</h2>
        <h3 className="flow-label">You → Agent</h3>
        <div className="flow-steps">
          <div className="flow-step">
            <div className="flow-icon">1</div>

[3] CLAIM: The single biggest message gap for “my agent can capture and read web pages” is that the agent is never the first actor in the conversion path — capture exists as a buried scenario and a late MCP card, not as a pasteable CLI recipe.
POINTER: index.tsx:79-97
OPENED TEXT:
<h3 className="flow-label">You → Agent</h3>
        <div className="flow-steps">
          <div className="flow-step">
            <div className="flow-icon">1</div>
            <h3>Paste or upload</h3>
            <p>Screenshot, file, or URL</p>
          </div>
          <div className="flow-connector" />
          <div className="flow-step">
            <div className="flow-icon">2</div>
            <h3>AI analyzes</h3>
            <p><T>OCR</T> + smart instructions</p>
          </div>
          <div className="flow-connector" />
          <div className="flow-step">
            <div className="flow-icon">3</div>
            <h3>Agent reads</h3>
            <p>Via <T>MCP</T> or <T>URL</T></p>
          </div>
        </div>
      </section>

      {/* ── Agent to Agent ────────────────────────── */}

[4] CLAIM: Codex and Grok are absent from the landing; the only install artifact is a Claude MCP add that requires `YOUR_KEY`.
POINTER: index.tsx:181
OPENED TEXT:
<h2>One command to connect</h2>
        <pre><code>{`claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header "X-API-Key: YOUR_KEY" -s user`}</code></pre>
        <p className="install-note">
          <a href="/login">Create an account</a> to get your API key.
          50 uploads/day, OCR, AI analysis, and full MCP integration - no credit card needed.

[5] CLAIM: “No account needed” sits on the same page as two account walls (MCP key; URL capture login), which contradicts a zero-click trial and the no-signup honesty rule.
POINTER: index.tsx:72
OPENED TEXT:
<UploadForm />
        <p className="quick-drop-hint">No account needed. Expires in 24 hours. Free.</p>
      </section>

      {/* ── How It Works - visual flow ─────────────── */}

[6] CLAIM: “No account needed” sits on the same page as two account walls (MCP key; URL capture login), which contradicts a zero-click trial and the no-signup honesty rule.
POINTER: index.tsx:183-184
OPENED TEXT:
<p className="install-note">
          <a href="/login">Create an account</a> to get your API key.
          50 uploads/day, OCR, AI analysis, and full MCP integration - no credit card needed.
        </p>
      </section>

[7] CLAIM: “No account needed” sits on the same page as two account walls (MCP key; URL capture login), which contradicts a zero-click trial and the no-signup honesty rule.
POINTER: UploadForm.tsx:182-185
OPENED TEXT:
// ── URL capture path ──
      if (!authenticated && !authLoading) {
        setError("URL capture requires an account. Please log in first.");
        return;
      }

      setState("uploading");
      try {

[8] CLAIM: “No account needed” sits on the same page as two account walls (MCP key; URL capture login), which contradicts a zero-click trial and the no-signup honesty rule.
POINTER: UploadForm.tsx:495-499
OPENED TEXT:
</p>
          {!authLoading && !authenticated && (
            <div className="url-auth-notice">
              <p>URL capture requires an account.</p>
              <a href="/login" className="btn btn-secondary">Log in</a>
            </div>
          )}
        </div>
      )}

[9] CLAIM: The hero drop-zone sells “enter a URL” as equivalent to dropping a file, without disclosing the login wall until after a URL is detected.
POINTER: UploadForm.tsx:405
OPENED TEXT:
<div className="drop-zone">
          <p>Drop files, paste a screenshot, or enter a URL</p>
          <div className="drop-zone-actions">
            <input
              ref={fileInputRef}

[10] CLAIM: The hero drop-zone sells “enter a URL” as equivalent to dropping a file, without disclosing the login wall until after a URL is detected.
POINTER: UploadForm.tsx:469-499
OPENED TEXT:
{/* URL detected - shown inline when a URL is pasted/dropped */}
      {hasUrl && (
        <div className="url-detected">
          <div className="url-detected-header">
            <span className="url-detected-label">URL detected</span>
            <button className="url-detected-clear" onClick={clearUrl}>Clear</button>
          </div>
          <div className="url-detected-value">
            <code>{url}</code>
          </div>
          <div className="url-options-row">
            <label className="url-checkbox">
              <input
                type="checkbox"
                checked={allDevices}
                onChange={(e) => setAllDevices(e.target.checked)}
                disabled={isBusy}
              />
              <span>All devices</span>
            </label>
            <span className="url-devices-hint">
              {allDevices ? "Desktop + Tablet + Mobile" : "Desktop only"}
            </span>
          </div>
          <p className="url-hint">
            Captures a screenshot with OCR - works with Google Sheets, dashboards, docs, and any public page.
          </p>
          {!authLoading && !authenticated && (
            <div className="url-auth-noti

[11] CLAIM: The hero drop-zone sells “enter a URL” as equivalent to dropping a file, without disclosing the login wall until after a URL is detected.
POINTER: UploadForm.tsx:540
OPENED TEXT:
onClick={handleSubmit}
        disabled={isBusy || (!hasFiles && !hasUrl) || (hasUrl && !authenticated && !authLoading)}
      >
        {isBusy
          ? hasUrl ? "Capturing..." : state === "validating" ? "Converting..." : "Uploading..."

[12] CLAIM: First visit is a cookie clickwrap over the upload affordance on both viewports; declining is a hard product block, not analytics-only.
POINTER: CookieConsent.tsx:51-77
OPENED TEXT:
return (
    <div className="consent-overlay">
      <div className="consent-banner">
        <div className="consent-text">
          <h3>Before you continue</h3>
          <p>
            WhipNode uses cookies for essential functionality and anonymous
            analytics. By using this service, you agree to our{" "}
            <a href="/terms">Terms of Service</a> and{" "}
            <a href="/privacy">Privacy Policy</a>.
          </p>
          <p className="consent-detail">
            We collect anonymous usage data (page views, upload counts, format
            usage). No file content, notes, or personal information is sent to
            analytics. No cross-site tracking. No ads.
          </p>
        </div>
        <div className="consent-actions">
          <button className="btn btn-primary" onClick={accept}>
            Accept and continue
          </button>
          <button className="btn btn-secondary" onClick={decline}>
            Decline
          </button>
        </div>
      </div>
    </div>
  );
}

export function ConsentGate({ children }: { children: React.ReactNode }) {

[13] CLAIM: First visit is a cookie clickwrap over the upload affordance on both viewports; declining is a hard product block, not analytics-only.
POINTER: CookieConsent.tsx:83-90
OPENED TEXT:
if (consent === "declined") {
    return (
      <div className="consent-blocked">
        <h2>Cookies required</h2>
        <p>
          You must accept the <a href="/terms">Terms of Service</a> and cookie
          policy to use WhipNode. This is required for the service to function.
        </p>
        <button
          className="btn btn-primary"
          onClick={() => {

[14] CLAIM: First visit is a cookie clickwrap over the upload affordance on both viewports; declining is a hard product block, not analytics-only.
POINTER: __root.tsx:103-105
OPENED TEXT:
<ErrorBoundary>
                <ConsentGate>
                  <Outlet />
                </ConsentGate>
              </ErrorBoundary>
            </main>
            <footer className="footer">

[15] CLAIM: Hero “See how to scale” and the install line “Create an account to get your API key… 50 uploads/day… no credit card needed” point the trial ladder at signup/pricing instead of an anonymous first capture.
POINTER: index.tsx:47
OPENED TEXT:
<a href="/new" className="btn btn-primary">Upload something</a>
            <a href="/pricing" className="btn btn-secondary">See how to scale</a>
          </div>
        </div>

[16] CLAIM: Hero “See how to scale” and the install line “Create an account to get your API key… 50 uploads/day… no credit card needed” point the trial ladder at signup/pricing instead of an anonymous first capture.
POINTER: index.tsx:183-184
OPENED TEXT:
<p className="install-note">
          <a href="/login">Create an account</a> to get your API key.
          50 uploads/day, OCR, AI analysis, and full MCP integration - no credit card needed.
        </p>
      </section>

[17] CLAIM: Hero “See how to scale” and the install line “Create an account to get your API key… 50 uploads/day… no credit card needed” point the trial ladder at signup/pricing instead of an anonymous first capture.
POINTER: pricing.tsx:18-24
OPENED TEXT:
<p className="pricing-subtitle">
        WhipNode is in early access. All early adopters get the free plan - no limits will change
        without notice.
      </p>

      <div className="pricing-early-banner">
        Early access - all features included free while we build in public.
      </div>

      <div className="pricing-grid">
        <div className="plan-card plan-current">

[18] CLAIM: The trust chip “End-to-end encrypted” is an honesty risk against the page’s own server-side OCR path (the service must read content).
POINTER: index.tsx:189-196
OPENED TEXT:
{/* ── Trust Bar ─────────────────────────────── */}
      <section className="trust-bar">
        <span>Global edge network</span>
        <span className="trust-divider">&middot;</span>
        <span>24h auto-delete</span>
        <span className="trust-divider">&middot;</span>
        <span>End-to-end encrypted</span>
        <span className="trust-divider">&middot;</span>
        <span>Open MCP standard</span>
      </section>
    </div>
  );

[19] CLAIM: The trust chip “End-to-end encrypted” is an honesty risk against the page’s own server-side OCR path (the service must read content).
POINTER: index.tsx:86-88
OPENED TEXT:
<div className="flow-connector" />
          <div className="flow-step">
            <div className="flow-icon">2</div>
            <h3>AI analyzes</h3>
            <p><T>OCR</T> + smart instructions</p>
          </div>
          <div className="flow-connector" />

[20] CLAIM: The trust chip “End-to-end encrypted” is an honesty risk against the page’s own server-side OCR path (the service must read content).
POINTER: UploadForm.tsx:180-205
OPENED TEXT:
if (hasUrl) {
      // ── URL capture path ──
      if (!authenticated && !authLoading) {
        setError("URL capture requires an account. Please log in first.");
        return;
      }

      setState("uploading");
      try {
        events.uploadStarted({ file_count: 1, source: "url_capture" });
        const resp = await captureUrl({
          url: url.trim(),
          ...(allDevices ? { viewports: ["desktop", "tablet", "mobile"] } : {}),
          note: note || undefined,
        });
        setResult({
          id: resp.id,
          share_url: resp.url,
          json_url: resp.json_url,
          markdown_url: resp.markdown_url,
          yaml_url: resp.prompt_url,
          txt_url: resp.markdown_url.replace(/\.md$/, ".txt"),
          expires_at: resp.expires_at,
        });
        setState("uploaded_processing");
        events.uploadCompleted({ item_id: resp.id, file_count: 1, total_bytes: 0 });
      } catch (err) {
        const msg = err instanceof Error ? err.message : "Capture failed.";
        setError(msg);

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }