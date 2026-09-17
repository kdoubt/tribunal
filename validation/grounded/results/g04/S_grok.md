I'll ground this in the capture pack and the landing-page source, then take an independent position on whether the page speaks to the agent-CLI adopter.The landing source is in hand; next I’ll confirm the upload CTA, cookie gate, MCP install copy, and pricing against the capture.I’ll check MCP install, FAQ, and onboarding copy so claims about signup, Codex/Grok, and capture stay tied to the source.**Framings considered:** (A) JTBD messaging — does the page sell “my agent captures and reads pages,” or “I upload for Claude”? (B) Funnel/auth integrity vs the zero-click trial ladder. (C) Honesty of trust/pricing claims. (D) Mobile parity of the one CLI install block. **I take (A) as primary** because Q1 is the decision; (B)–(D) answer Q2/Q3 as consequences of that mismatch.

---

CLAIM: The landing’s primary voice is a human uploader unblocking Claude Code, not a terminal agent that captures pages itself.
EVIDENCE: Pack: “You can't paste screenshots into Claude Code. WhipNode fixes that.” Confirmed `index.tsx:42-47` (`hero-explainer`, CTAs `Upload something` → `/new`, `See how to scale` → `/pricing`) and `index.tsx:78` (`You → Agent`).
CONFIDENCE: 0.9
FALSIFIER: A first-screen headline/CTA whose subject is the agent capturing a URL, with human upload demoted below the fold.

CLAIM: The single biggest message gap for “my agent can capture and read web pages” is that the agent is never the first actor in the conversion path — capture exists as a buried scenario and a late MCP card, not as a pasteable CLI recipe.
EVIDENCE: Pack: “How it works / You - Agent / Paste or upload”; “Capture e revieu / One agent screenshots staging” (a2a, not hero); “6 MCP tools… Fetch, upload, screenshot… Native to Claude Code”; install-note “Create an account to get your API key.” Source: `index.tsx:79-97`, `127-129`, `170-185`. No recipe/trial CTA in `index.tsx` (only `/new` and `/pricing`).
CONFIDENCE: 0.85
FALSIFIER: Above-fold copy plus a one-liner that makes `whipnode_capture` / equivalent the first action for Claude Code, Codex, and Grok with no human upload.

CLAIM: Codex and Grok are absent from the landing; the only install artifact is a Claude MCP add that requires `YOUR_KEY`.
EVIDENCE: Pack: `claude mcp add whipnode -transport http https://whipnode.com/api/v1/mcp -header *x-AF`. Source: `index.tsx:181` `` `claude mcp add whipnode --transport http … --header "X-API-Key: YOUR_KEY" -s user` ``. `index.tsx` contains no `Codex` or `Grok` string.
CONFIDENCE: 0.95
FALSIFIER: Visible Codex/Grok connect copy or a non-Claude one-liner on `/` in capture or `index.tsx`.

CLAIM: “No account needed” sits on the same page as two account walls (MCP key; URL capture login), which contradicts a zero-click trial and the no-signup honesty rule.
EVIDENCE: Pack: “No account needed. Expil” vs “Create an account to get your API key.” Source: `index.tsx:72` “No account needed. Expires in 24 hours. Free.” vs `index.tsx:183-184`; `UploadForm.tsx:182-185` `if (!authenticated …) { setError("URL capture requires an account. Please log in first."); return; }` and `UploadForm.tsx:495-499` “URL capture requires an account.”
CONFIDENCE: 0.92
FALSIFIER: Capture/MCP usable with no account, or the landing labels which verbs need login vs which do not.

CLAIM: The hero drop-zone sells “enter a URL” as equivalent to dropping a file, without disclosing the login wall until after a URL is detected.
EVIDENCE: Pack: “Drop files, paste a screenshot, or enter a URL” / “Brouse Files or ps://docs.90ogle.con/”. Source: `UploadForm.tsx:405` same drop-zone copy; auth notice only in the `hasUrl` branch (`UploadForm.tsx:469-499`); submit disabled when `hasUrl && !authenticated` (`UploadForm.tsx:540`).
CONFIDENCE: 0.9
FALSIFIER: Drop-zone copy that states URL capture needs an account (or trial), or unauthenticated URL capture succeeding from `/`.

CLAIM: First visit is a cookie clickwrap over the upload affordance on both viewports; declining is a hard product block, not analytics-only.
EVIDENCE: Pack desktop+mobile: “Before you continue / WhipNode uses cookies… Accept and continue / Decline” adjacent to “Task for the” / “0/4000”. Source: `CookieConsent.tsx:51-77` overlay; `CookieConsent.tsx:83-90` on decline: “You must accept the Terms of Service and cookie policy to use WhipNode. This is required for the service to function.” `__root.tsx:103-105` wraps the route in `ConsentGate`.
CONFIDENCE: 0.85
FALSIFIER: Capture of a first visit with no overlay on the form, or decline still allowing upload/MCP.

CLAIM: Hero “See how to scale” and the install line “Create an account to get your API key… 50 uploads/day… no credit card needed” point the trial ladder at signup/pricing instead of an anonymous first capture.
EVIDENCE: Pack: “See how to scale”; “Create an account to get your API key.50 uploads/day… no credit card needed.” Source: `index.tsx:47` `href="/pricing"`; `index.tsx:183-184`. Pricing page (out of primary artifact, supporting): `pricing.tsx:18-24` early-access free plan — not shown on `/`.
CONFIDENCE: 0.75
FALSIFIER: Second hero CTA is a trial/recipe action, and install copy describes no-email connect with trial quota.

CLAIM: The trust chip “End-to-end encrypted” is an honesty risk against the page’s own server-side OCR path (the service must read content).
EVIDENCE: Pack: “Gtobat edge netmork24 auto-deteteEnd-to-end encrypted open ncp standard.” Source: `index.tsx:189-196` `End-to-end encrypted`; `index.tsx:86-88` step 2 “AI analyzes / OCR + smart instructions.” Upload posts file/URL content to the API (`UploadForm.tsx:180-205`, `256-264`).
CONFIDENCE: 0.7
FALSIFIER: A documented client-side E2E scheme where WhipNode cannot read bytes/OCR, or the chip removed/qualified to transport TLS only.

---

**TOP-3 CHANGES**
1. Rewrite hero + primary CTA to agent-first capture (“Your agent can screenshot any URL”) with Claude / Codex / Grok one-liners; demote “Upload something” and drop “See how to scale” from the hero.
2. Make auth copy match the trial: replace “Create an account to get your API key” with the real no-email connect; restrict “No account needed” to file paste, and disclose the URL-capture gate in the drop-zone.
3. Strip or qualify “End-to-end encrypted”; make cookie non-blocking (no `ConsentGate` hard wall); add a visible recipe-page link for the review-methodology adopter.

**VERDICT INPUT**
- **Q1:** Speaks to the human Claude-Code uploader first; agent-CLI capture/read is a mid-page scenario. Biggest gap: no agent-initiated capture recipe on the conversion path.
- **Q2:** Ship the three changes above (hero/CLI one-liners, honest trial auth, trust/cookie/recipe) before ladder launch — all one-person copy/CTA work in `index.tsx` + `UploadForm`/`CookieConsent`.
- **Q3:** Yes — “No account needed” vs API-key/login, cookie hard-block on decline, E2E chip, and a hero path into Pricing all fight a zero-click no-signup ladder.
