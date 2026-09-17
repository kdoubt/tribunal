1. **ATTACK**

- **Peer-1:** The schema constraint is established; “needs a synthetic sentinel” does not follow. Nullable email plus explicit trial state is another implementation. A “plain UPDATE” is insufficient: an existing email collides with uniqueness, and verification must bind the email, trial account and approving installation before conversion. `auth-verify.ts:39-50` selects accounts by email; it does not implement that transaction. Ladder step 10 also contradicts itself by setting `plan='free'` while claiming the account remains trial. Confidence 0.93 overstates the proposed remedy.
- **Peer-2:** Correct about the configured threshold. “Grants 50/day” must mean the fallback allowance, not an enforced maximum: the settled admission race remains. No substantive disagreement with the lookup finding.
- **Peer-3:** Correct that anonymous enrollment requires code changes and credentials must stay outside tool text. “Must ride” `Mcp-Session-Id` is unsupported exclusivity: it is a reusable transport, not proof of installation possession. `middleware/auth.ts:30-33,72-83` authenticates whoever presents that token. Header placement alone also needs client logging/redaction verification.
- **Peer-4:** TTL and single use do **not** defeat hijack (a): a transcript reader can redeem first. Ladder step 8 puts both URL and possession code into the same leaked surface, making their combination a bearer credential. For (b), the attacker can supply their own URL **and code**; the server cannot establish that a manually entered code came from the victim’s installation. Activity display is a warning, not binding proof. “Only a code” is unjustified; authenticated local possession is an alternative. The falsifier is inadequate: removing email input from this page could merely move the vulnerable attachment downstream.
- **Associated ladder:** Preserving attacker-held keys and every device row preserves attacker access after victim verification. Email confirmation proves mailbox control, not installation ownership. Reusing an account by IP/UA also cannot establish possession: unrelated users can share both. Finally, an issuance budget is not a daily compute ceiling for accumulating, renewable trials; the proposed viewport restriction supplies neither a total deadline nor atomic work reservations.

2. **CONCEDE**

The final answer must retain the explicit trial-plan definition, real anonymous-enrollment change, existing session-header transport as a reuse option, and atomic single-use consumption as one component of claim security. A sentinel is technically possible, although nullable email is preferable. Preserve account/item identity where possible; that does not require preserving credentials. None of these changes establishes “zero PII”: deterministic IP/UA hashes and captured content remain unresolved.

3. **REVISE**

- **Own-5:** New text: “Originating-device approval or a transcript-visible code cannot establish victim-installation binding; require a browser-session-bound proof from the victim’s installation, with relay resistance demonstrated before launch.” Confidence **0.95 → 0.92**. My loopback proposal is a candidate, not a demonstrated solution; Host/Origin checks alone do not establish the full guarantee.
- **Own-7:** New text: “Rung 0 complements `init`; reuse enrollment and token-consumption components, while conversion preserves account/items and carries only the proven device after replacing its credentials and revoking other trial access.” Confidence **0.88 → 0.90**. Device authentication checks the parent key’s status (`middleware/auth.ts:103-104`), so carrying a row requires updating its key association.

4. **VERDICT INPUT**

**Q1:** Capped pilot only after atomic attempt budgets, enforced resource bounds, and an independent trial admission switch; abuse profitability and literal zero PII remain unproven.

**Q2:** Reject the leaked-code design; use a public locator plus verified victim-installation/browser binding before email attachment and explicit Authorize.

**Q3:** Complement `init`; reuse approval/enrollment components, preserve live items, replace credentials, and advertise honest quotas and unchanged free retention.
