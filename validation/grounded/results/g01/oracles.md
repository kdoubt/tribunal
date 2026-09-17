# Oracles — g01 (mechanical pointer resolution + inspection; orchestrator never rates "support")

grounding.py: S_claude 13/13 pointers open; S_grok 29/29 pointers open (S_claude.grounding.json, S_grok.grounding.json).
Every cited span was opened by hand (sed) after resolution. Findings by inspection:

- mcp.ts:333-353, :386 — `if (auth.type === "api_key" && isInitializeRequest(body))` … conditional INSERT into mcp_devices `WHERE (SELECT COUNT(*) … status='active') < 10` … `headers.set("Mcp-Session-Id", sessionToken)`; expiresAt = 90 days. Settles A1, B3 (mechanism), A5/B4 (10-device cap), B7's "today 10/key".
- auth-verify.ts:20-24 — `UPDATE magic_tokens SET used_at = ? WHERE token_hash = ? AND used_at IS NULL AND expires_at > ?` (A step 9 idiom exists). :43-50 — the only `INSERT INTO accounts … 'free'` (B2). :72-77 — `Location: ${env.APP_BASE_URL}/dashboard` + `Set-Cookie: wn_session=` (A2).
- env.ts:3-7 — bindings DB, R2, QUEUE, BROWSER, AI; no KV (A2).
- auth-magic-link.ts:22-48 — `WHERE email = ? AND created_at > ?` count ≥3 → RATE_LIMITED; ipHash computed (:41) and bound (:47); no query reads ip_hash in this file (A3, B7).
- grep -rniE 'turnstile|captcha|hcaptcha|recaptcha' over the artifact → 0 files (A4, B7). login.tsx:72-89 = bare email form (A4). Cloudflare WAF/Turnstile config is not in the tree — SPECULATIVE as A labelled.
- constants.ts:42-62 — free: uploads_per_day 50, 25 MiB item/file, ttl 24, api_access true, max_api_keys 1, history false; pro: 1000, 250 MiB item, 50 MiB file, ttl 168, 10 keys, history true (A5, A7, B4, B5).
- api-keys.ts:40-60 — `>= maxKeys` → FORBIDDEN "Revoke an existing key or upgrade."; INSERT scopes '["write"]'; raw key returned once (B4).
- capture.ts:227-236 — `if (!auth) … "URL captures require an API key."`; then `checkDailyUploadLimit` (A framing, A6, B1). :534-548 — viewports validated against VALID_VIEWPORTS, max 6; no plan gate (B5). :565 — `puppeteer.launch(env.BROWSER)` before the per-viewport loop (A6: "inside the per-viewport loop" is imprecise — launch is once per capture, the loop is over viewports at :567; the cost point stands: a browser per capture).
- pricing.tsx:25-55 — Early Access $0: 50 uploads/day, 25 MB, 24h, OCR, 1 API key; Pro TBD: "50 MB per file, 100 MB per item" (A7: 100 MB vs constants 250 MiB confirmed; B6: price TBD confirmed).
- mcp.ts:670-676 — status lines `Account: … (plan)` and `Uploads today: N`; no limit value (A8, B6). rate-limit.ts:80-87 — reason string names plan and limit only on refusal (A8, B6).
- mcp.ts:289-293 — unauth → jsonRpcError -32000 "Pass your API key in the X-API-Key header." (B1). docs.tsx:28-44 — Step 1 sign in, Step 2 create key in dashboard, Step 3 `claude mcp add … --header "X-API-Key: …"` (B1, A step 10). ROADMAP.md:36 "CLI tool for terminal workflows" under future items (B1).
- email.ts:22-33 — subject "Sign in to WhipNode"; no device/consent wording (B2).
- auth.ts:24-53 — Mcp-Session-Id resolved first, expired sessions fall through to X-API-Key (B4); :112-116 sliding 90-day expiry (B3, B8).
- ApiKeyManager.tsx:308-311 "Save this key now - it won't be shown again." (B4). terms.tsx:61 "No API access (browser and paste only)" vs login.tsx:94-95 "Free accounts include 1 API key" (B5 staleness confirmed).
- docs/private/ABUSE_PREVENTION.md:1-40 — IP-based limits (20 uploads/IP/h etc.), hashed IP, no Turnstile mention (B7).
- mcp.ts:212-218 — configure returns a PROPOSAL, apply only after user confirmation (B7 human-in-loop precedent). :282-284 revoked/expired copy (B8). :367-376 quota-hit path logs and returns without enrolling (B step 8 "bad pattern"). :699-703 group visibility lines (A trigger 4).
- mcp-devices.ts:68-75 revoke UPDATE; api-keys.ts:137-144 cascade revoke of devices (B8). PRIVACY.md:1-17 stored fields (B8). McpDeviceManager.tsx:77 "auto-renews on use (expires if inactive for 90 days)" (B8). items-create.ts:60-66 25 MB request cap with paid-plan pointer (B trigger 4).

Verified-by-inspection rows: A1, A2, A3, A4, A5, A6 (mechanism), A7, A8, B1, B2, B3, B4, B5 (facts), B6, B7 (facts), B8. Design choices inside those claims (caps, units, placement) remain PANELIST-CLAIM — nothing in the tree settles them.
