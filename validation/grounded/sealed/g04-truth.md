# G04 sealed rubric - SEALED (arms never see this)

- **difficulty:** control (both original seats agreed on everything decision-relevant at Round 0)
- **correct_call:** The page is human-uploader-first, not agent-first; three
  blockers before the trial-ladder launch: (1) an agent-first hero, (2) a
  multi-CLI install snippet (Claude Code, Codex, Grok), (3) remove the
  "Create an account to get your API key" contradiction with the zero-click
  trial. Also: the cookie wall covers the hero on both viewports and pricing
  is invisible on-page.
- **oracle:** all three blockers shipped with the 2026-08-20 launch and were
  live on whipnode.com on 2026-09-08 (agent-first hero "Your agent captures
  and reads any page"; multi-CLI install `claude mcp add whipnode -- npx
  whipnode mcp`; zero "Create an account"/"Sign up" strings; "No signup"/"No
  account needed" present). Source confirmation in the artifact:
  `apps/web/app/routes/index.tsx:41` (hero "Your AI agent can finally see."),
  `:72` ("No account needed. Expires in 24 hours. Free."),
  `:181` (install snippet names only `claude mcp add`),
  `:183-184` ("Create an account to get your API key. 50 uploads/day ... no
  credit card needed."), cookie banner component
  `apps/web/app/components/CookieConsent.tsx`, pricing reachable only via
  `:47` "See how to scale" link.
- **correct fix:** rewrite the hero around capture-and-read for agents; add
  Codex/Grok install lines; replace the account CTA with the zero-click path;
  move or shrink the cookie banner so the hero is readable; surface the free
  tier on-page.
- **must_catch:**
  1. Hero and primary CTA address a human uploader ("Upload something",
     "Drop files, paste a screenshot") - pack quotes; `index.tsx:41-47`.
  2. The install snippet is Claude-only; Codex and Grok are unnamed
     (`index.tsx:181`; pack "claude mcp add whipnode ...").
  3. Contradiction: "No account needed" (pack; `index.tsx:72`) versus
     "Create an account to get your API key" (pack; `index.tsx:183-184`) -
     collides with a zero-click trial.
  4. The cookie consent banner overlays the hero on both desktop and mobile
     captures (pack: "Before you continue ... Accept and continue" appears
     within the hero text in both dumps).
  5. Pricing is not on the page (only a "See how to scale" link; pack shows no
     prices); the agent-adopter cannot see the free tier without navigating.
  6. The page's agent-orchestration section ("Pastebin for agents" in the
     pack) describes agent-to-agent handoff scenarios but supplies no
     actionable trial recipe: no command, no key step, no first call an agent
     could run (pack; `index.tsx` has an install command at `:181` but no
     recipe or first-call block).
  7. OCR noise ("somethins", "Brouse Files", "stooing-cyopp") must be labelled
     NO-SIGNAL, not reported as typos on the page.
- **landmine (confident wrong answers):** reporting the OCR garble as real
  copy defects; "the page already targets agents because it says 'Your AI
  agent can finally see'" (the CTA flow beneath is upload-first);
  "pricing is visible" (it is a link).
- **unverifiable:** mobile layout beyond what the OCR ordering shows.
- **outcome_source:** `2026-08-19-whipnode-landing-dogfood/retro.md`
  (agreed_r0 list; T1 outcome: all three blockers live, checked 2026-09-08;
  verdict_held yes; missed_entirely none).
