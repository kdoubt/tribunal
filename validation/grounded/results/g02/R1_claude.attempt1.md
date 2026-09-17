## 1. ATTACK

**Peer-1 (complement + local-adapter binding).** The headline claim is settled and fine at 0.88; its *ladder spec* is where it fails. Step 9 — a local MCP adapter holding a signing key, challenged by the claim page "over loopback" — is a new component with its own key management, and it breaks exactly where rung 0 matters most: MCP servers running in IDE devcontainers, SSH remotes, or hosted agents have no browser-reachable loopback. Peer-1 concedes this ("unsupported environments use explicit `init`"), which means the zero-click trial is unclaimable for a large share of its own audience — the ladder dead-ends at rung 1. Against the "no new services / existing flows preferred" constraint, that is a heavier lift than a 6-char code the CLI already prints. Its step 6 figure ("100 attempts/day ≈ 2,500 browser-seconds") also silently assumes the 1-viewport fix; unfixed, `capture.ts` allows 6 viewports × 28 s and the browser is closed only on the success path, so the true ceiling is leak-unbounded. The number is right only as a post-fix target, not as a budget you can set today.

**Peer-2 (zero-PII).** Correct and the strongest claim on the board. No attack.

**Peer-3 (hijack b).** Right conclusion, but its falsifier prescribes the wrong cure. "Proof from the installation connected to the victim's browser" does not defeat (b): an attacker who originates the link also holds that installation and passes any possession proof. Peer-3's own control defeats the attack it names only if the victim's *email verification* is the gate — which Peer-3 lists as an afterthought ("bound to that same email-verification session") rather than the load-bearing control. 0.95 is fine on the negative claim; the constructive half is under-argued.

**Peer-4 (no stable position).** This is an abstention, not a claim, and it declines the question Q1 actually asks. The cost side is computable from the artifact without knowing attacker economics: worst case ≈168 browser-seconds + 6 OCR jobs per capture, and a global daily trial budget converts that into a hard monetary ceiling the owner sets. You do not need measured attacker acquisition costs to bound your own downside — you need a budget. Its constants evidence is a restatement of settled ORCH material and adds nothing at 0.85.

**Peer-5 (honest upsell).** Correct; free is also 24 h / `history:false`, so the delta is 10→50/day and nothing else. Note the trap my CLAIM 6 identifies: without adding `PLAN_LIMITS.trial`, the trial already gets 50/day and the upsell is *zero*.

**Peer-6 (quotas as admission control).** Correct and the finding I under-weighted. Check-at-:234 / insert-at-:663 with `Promise.all` batches at mcp.ts:319-320 means N concurrent captures all pass a stale count, and failed captures burn compute uncharged.

## 2. CONCEDE

- Peer-2: "zero PII" is false as stated — deterministic IP/UA hashes over a 2³² space are re-identifiable. Say "no email, pseudonymous network metadata, disclosed."
- Peer-6: quota checks must become an atomic reserve-before-launch, charging failures.
- Peer-5: copy must state that claiming restores nothing expired.
- Peer-3: device possession alone does not defeat (b).

## 3. REVISE

**CLAIM 8** → "TTL + single-use defeat hijack (a); for (b) the load-bearing control is that `plan` flips only after the attached email verifies via magic link — the CLI code is a secondary speed bump, since an attacker can relay their own code to a victim." Confidence 0.8 → **0.86**.

**CLAIM 5** unchanged but re-scoped: sentinel email is synthetic, not PII; Peer-2's objection lands on IP/UA hashes, not here. 0.93 held.

Spec amendment: step 5 becomes an atomic D1 reservation (`UPDATE ... WHERE remaining > 0`) before `browser.launch`, charging failed attempts; add a `finally` close for capture.ts:622.

## 4. VERDICT INPUT

- **Q1:** Survivable only after three fixes — atomic pre-launch reservation, trial capped at 1 viewport / no full-page / no 4k, and browser close in `finally` — behind a global daily trial budget the owner can zero without deploy; not survivable on today's code.
- **Q2:** Public-locator link + 15-min single-use token; claim page shows recent-activity summary and takes a CLI-displayed code; plan flips only on verified email — no loopback adapter.
- **Q3:** Complements `init`; reuse enrollment and the atomic magic-token consume; same `account_id`, same key, `mcp_devices` carried; recipe page gains "Try without email" alongside `init`.
