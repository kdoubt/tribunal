I'll re-open the spec and the cited worker/schema pointers so the attacks land on evidence, not on Round 0 memory.I’ll check the cited worker, schema, and spec lines against each peer claim before attacking or conceding.1. **ATTACK**

**Peer-1 (0.96):** The raw-key contradiction is real (`S:68-72` vs `S:202-204`). Confidence is too high: “trial quota remains explicitly open” fights their own 3-device reading — `:94` puts 10 in the shipping table and `:136` hardcodes `10/10` in wall copy, the same “defaults ship” move as `:240-241`. Bot-score is tagged OWNER-CALL at `:282` with env default 30, not an unlabeled product add; the real extra is the ASN deny-list that “must be built.” `:202-204` is about what *init* delivers; `:68-72` is trial transport. They collide only via “must follow the same choice.”

**Peer-2 (0.94):** Post-conversion key persistence is right; the walkthrough is not. Conversion’s WHERE re-asserts `plan='trial'` (`S:187`). A thief cannot “start a replacement claim, verify their own unused mailbox, authorize, and poll” on an already-free account — that poll 409s. Never-merge (`S:177-181`) also blocks rebinding. Continuing access is the still-active stolen key, not a second claim. 0.94 treats a non-specified re-claim as fact.

**Peer-3 (0.98):** FK-blocked `DELETE` is right: D1 enforces FKs by default; revoke is `UPDATE` status (`api-keys.ts:136-143`); `api_keys`/`mcp_devices`/`items` FKs remain. 0.98 still bundles three failures: (1) those child FKs, (2) the settled `cleanup.ts:22-24` early return, (3) unspecified `device_codes` purge for a table the spec itself creates — a missing task, not an existing-FK failure. `api-keys.ts` is the dashboard revoke path, not the cleanup task.

**Peer-4 (0.96):** Render is outside the inventory; Browser launch with six viewports + `full_page` and no quota (`render.ts:56-66,:118-141`) is a real spend-ceiling bypass. Framing overreach: this is a pre-existing HTTP route, not a hole “the spec’s own protocol introduces.” `/api/v1/uploads` is an equal unlisted authenticated spend path.

2. **CONCEDE**

- Rung 0 operationally selects raw `wn_` + `X-API-Key` (`S:68-72,:274`) while `:202-204` still says UNRESOLVED; an agent will ship raw keys.
- Stolen 0600 remains a full bearer after conversion (`S:189`; `auth.ts:174-192`); 24h item TTL does not bound post-claim access.
- `DELETE accounts` after revoke-in-place fails D1 FKs on remaining `api_keys`/`mcp_devices` (and tombstoned `items`).
- `/api/v1/render` is absent from trial spend/viewport controls.
- Ladder core stands: complementary init (`S:22`) and same-key conversion (`S:185-189`).
- Settled R0 blockers stand: email rebuild, session-authed approve, `trial_counters` UPDATE-only, cleanup early return, uncomposed `device_codes`.

3. **REVISE**

**Own-1** (0.86→0.90): The spec encodes both USER-FACT decisions but operationally selects raw `wn_` header/`X-API-Key` delivery (`S:68-72`) while labeling key-vs-token UNRESOLVED (`S:202-204`), ships quota 10 and 3 devices as defaults, and adds an ASN deny-list neither decision made; only Turnstile is truly open.

**Own-5** (0.91→0.93): Named 7-day cleanup both sits behind `cleanup.ts:22-24`’s empty-item return and, if reached, cannot `DELETE` the account: revoke only UPDATEs key/device status, child FKs remain, D1 enforces FKs by default.

**Own-7** (0.78→0.86): Stolen 0600 is a complete pre-claim credential; after conversion the same active key is a full free-account bearer, so 24h TTL does not bound continuing access; freeze with no `expires_at>now` still lets claim-start refresh the 7-day clock. Post-conversion mailbox-rebind via a new claim is not in the spec (`S:187` requires `plan='trial'`).

Own-2, Own-3, Own-4, Own-6, Own-8: no revision.

4. **VERDICT INPUT**

Q1 Fidelity: Encodes both decisions; silently ships 10/3, operationally picks raw keys, adds ASN/bot gates.  
Q2 Buildability: Unbuildable — email rebuild, approve-without-session, counters without INSERT, cleanup/FKs, no `purpose` column, no per-IP table, render unlisted.  
Q3 Security: 0600 is the claim secret and persists post-conversion; freeze unbounded; render/uploads bypass the spend ceiling; locator-only is not a new bearer.  
Q4 Traps: wrapper mint signal unnamed; two machines on `device_codes` uncomposed; note-parsed snapshot lossy; response header unnamed.  
PUBLISH VERDICT: `blocker` — an implementing agent cannot execute the migrations, approve path, counters, or cleanup as written, and would guess raw-key delivery and leave render unconstrained.
