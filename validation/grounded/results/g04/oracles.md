# Oracles — g04 (orchestrator, mechanical; no support ratings)

grounding.py: grok 20/20 pointers open. codex uses shorthand (`R:` = index.tsx, `U:` = UploadForm.tsx, `C:` = CookieConsent.tsx, stated in its memo) so the resolver found 0; resolved by hand: R:47, R:72, R:163, R:181, R:183-184, R:194, U:182-184, U:540, C:32-34 — all open and match.

Pack quotes: every quoted string from both seats exists in the brief's inlined pack (hero line is wrapped across two lines; grok's "Gtobat edge netmork24 auto-deteteEnd-to-end encrypted" is the verbatim OCR line 143).

Verified by inspection (cited text plainly settles the claim):
- index.tsx:41-47: h1 "Your AI agent can finally see.", explainer "You can't paste screenshots into Claude Code.", CTAs `/new` "Upload something" and `/pricing` "See how to scale" — A1, A7 mechanism, B7 mechanism.
- index.tsx:72 "No account needed. Expires in 24 hours. Free." vs :183-184 "Create an account to get your API key. 50 uploads/day…" — A4, B2, A7/B7 facts.
- index.tsx:181 `claude mcp add whipnode … --header "X-API-Key: YOUR_KEY" -s user`; no "Codex"/"Grok" string in index.tsx — A3, B3 facts.
- UploadForm.tsx:182-184 sets "URL capture requires an account. Please log in first." and returns; :495-499 the auth notice inside the `hasUrl` block; :540 submit disabled when `hasUrl && !authenticated`; :405 drop-zone copy "Drop files, paste a screenshot, or enter a URL" — A4, A5, B2.
- CookieConsent.tsx:32-34 prompt shown when stored consent is "unknown"; :51-77 overlay with Accept/Decline; :83-90 `ConsentGate` on decline renders "Cookies required … required for the service to function"; __root.tsx:103-105 wraps `<Outlet/>` in `ConsentGate` — A6's hard block on the web route is verified; B6's "SPECULATIVE for the terminal path" stands (the gate wraps the web app routes; the MCP/API path is not in this tree's routes).
- index.tsx:194 "End-to-end encrypted"; :86-88 "AI analyzes / OCR + smart instructions" — A8/B5 facts.
- index.tsx:163 "Desktop, tablet, or mobile viewport." — B4's quoted line exists; the claim's premise (the Claude Code limitation is unverified) is not settleable from the artifact.
- pricing.tsx:18-24 early-access free plan text — A7's supporting fact.
Not settleable here: whether the cookie gate affects the MCP/terminal path (no server route in the artifact); mobile terminal readability (B8 NO-SIGNAL).

## Round 1 addendum (2026-09-17)
- No new `file:line` was cited in either Round 1 answer; both rebuttals reuse `CookieConsent.tsx` / `ConsentGate` and `index.tsx:47,:181,:183-184`, all opened above.
- Scope check raised by B on A7: the brief names the inlined pack as primary and `apps/web/app/routes/index.tsx` plus `apps/web/app/components/` as the only "secondary, in bounds" sources. `apps/web/app/routes/pricing.tsx:18-24` lies inside the artifact root but outside those named paths. It was recorded in Round 0 as *supporting* only; A7's claim rests on `index.tsx:47` and `:183-184`, which are in bounds and verified. Kept as supporting, flagged; not relayed further.
- Post-R1 claims marked `verified` by inspection: none new. A6/B6's converged web-scoped statement is already covered by the `ConsentGate` check above.
