You are checking whether cited source text supports a claim. For each numbered item, answer 1 if the OPENED TEXT, read literally, supports the CLAIM (the claim's assertion about that location is true of that text), else 0. A pointer that opens to unrelated or contradicting text is 0. Do not use outside knowledge; do not evaluate whether the claim is a good idea.

[1] CLAIM: The page does speak to an agent-CLI adopter, but exclusively a Claude Code one — the landing route names Claude Code six times and names no other agent CLI, so a Codex/Grok arrival sees no proof it applies to them.
POINTER: faq.tsx:79
OPENED TEXT:
{
    q: "How do I use WhipNode with Codex or other AI agents?",
    a: "Upload your context at whipnode.com or via the API (POST /api/v1/items). Pass the JSON or prompt pack URL to your agent. Any agent that can make HTTP GET requests can consume WhipNode outputs - no SDK or special integration required. The API returns all format URLs in the creation response.",
  },
  {

[2] CLAIM: The page does speak to an agent-CLI adopter, but exclusively a Claude Code one — the landing route names Claude Code six times and names no other agent CLI, so a Codex/Grok arrival sees no proof it applies to them.
POINTER: docs.examples.tsx:51
OPENED TEXT:
<section>
        <h2>Codex Integration</h2>
        <pre><code>{`# Upload context before invoking Codex:
ITEM=$(curl -s -X POST https://whipnode.com/api/v1/items \\
  -H "Content-Type: application/json" \\

[3] CLAIM: The page does speak to an agent-CLI adopter, but exclusively a Claude Code one — the landing route names Claude Code six times and names no other agent CLI, so a Codex/Grok arrival sees no proof it applies to them.
POINTER: about.tsx:33
OPENED TEXT:
<p>
        AI coding agents like Claude Code, Codex, and Copilot can't read your
        screenshots. They can't access files on your clipboard. They can't parse
        the error dialog you're staring at. The context is trapped on your
        screen and there's no standard way to get it into an agent's context

[4] CLAIM: The page does speak to an agent-CLI adopter, but exclusively a Claude Code one — the landing route names Claude Code six times and names no other agent CLI, so a Codex/Grok arrival sees no proof it applies to them.
POINTER: __root.tsx:18
OPENED TEXT:
content:
          "Give your AI coding agent vision. Upload screenshots, paste URLs, drop files - get structured context via MCP with OCR, AI analysis, and responsive previews. Works with Claude Code, Codex, and any MCP-compatible tool.",
      },
    ],
    links: [

[5] CLAIM: The page makes a "No account needed" promise directly under an upload form whose URL path refuses anonymous users, which is a hard contradiction for a zero-click trial funnel.
POINTER: index.tsx:72
OPENED TEXT:
<UploadForm />
        <p className="quick-drop-hint">No account needed. Expires in 24 hours. Free.</p>
      </section>

      {/* ── How It Works - visual flow ─────────────── */}

[6] CLAIM: The page makes a "No account needed" promise directly under an upload form whose URL path refuses anonymous users, which is a hard contradiction for a zero-click trial funnel.
POINTER: UploadForm.tsx:182-185
OPENED TEXT:
// ── URL capture path ──
      if (!authenticated && !authLoading) {
        setError("URL capture requires an account. Please log in first.");
        return;
      }

      setState("uploading");
      try {

[7] CLAIM: The trust bar asserts "End-to-end encrypted," which the repo's own security doc lists as explicitly out of scope — an honesty-rule violation and the most defensible thing a skeptical developer could call out publicly.
POINTER: index.tsx:194
OPENED TEXT:
<span className="trust-divider">&middot;</span>
        <span>End-to-end encrypted</span>
        <span className="trust-divider">&middot;</span>
        <span>Open MCP standard</span>
      </section>

[8] CLAIM: The trust bar asserts "End-to-end encrypted," which the repo's own security doc lists as explicitly out of scope — an honesty-rule violation and the most defensible thing a skeptical developer could call out publicly.
POINTER: docs/SECURITY.md:51-52
OPENED TEXT:
### What is NOT in scope
- End-to-end encryption (context must be readable by agents)
- DRM or access revocation
- Permanent deletion guarantees beyond storage cleanup

[9] CLAIM: A blocking cookie modal is interposed over the hero and upload form on first paint in both viewports, and declining it disables the product entirely — the worst possible first rung of a zero-click ladder.
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

[10] CLAIM: The secondary hero CTA "See how to scale" routes to a pricing page that offers nothing to scale to, burning the page's second-most-valuable click.
POINTER: index.tsx:47
OPENED TEXT:
<a href="/new" className="btn btn-primary">Upload something</a>
            <a href="/pricing" className="btn btn-secondary">See how to scale</a>
          </div>
        </div>

[11] CLAIM: The secondary hero CTA "See how to scale" routes to a pricing page that offers nothing to scale to, burning the page's second-most-valuable click.
POINTER: pricing.tsx:18
OPENED TEXT:
<p className="pricing-subtitle">
        WhipNode is in early access. All early adopters get the free plan - no limits will change
        without notice.
      </p>

[12] CLAIM: The secondary hero CTA "See how to scale" routes to a pricing page that offers nothing to scale to, burning the page's second-most-valuable click.
POINTER: pricing.tsx:29
OPENED TEXT:
<h2>Early Access</h2>
          <p className="plan-price">$0</p>
          <ul>
            <li>50 uploads/day</li>
            <li>25 MB per item</li>

[13] CLAIM: The install command is the page's key conversion asset but is gated behind account creation, so the landing page currently has no zero-click entry point at all.
POINTER: index.tsx:181-184
OPENED TEXT:
<h2>One command to connect</h2>
        <pre><code>{`claude mcp add whipnode --transport http https://whipnode.com/api/v1/mcp --header "X-API-Key: YOUR_KEY" -s user`}</code></pre>
        <p className="install-note">
          <a href="/login">Create an account</a> to get your API key.
          50 uploads/day, OCR, AI analysis, and full MCP integration - no credit card needed.
        </p>
      </section>

OUTPUT (JSON only): {"items": [ {"n": 1, "supports": 0 or 1, "why": "<one clause>"}, ... ] }