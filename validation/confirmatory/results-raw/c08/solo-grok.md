# Decision Memo — C08: Is SameSite=Lax enough against CSRF?

**CALL:** No. `SameSite=Lax` is a useful brake, not a CSRF control. Keep a per-request anti-CSRF check (synchronizer token, double-submit, or Fetch Metadata `Origin`/`Sec-Fetch-Site` allowlist) on every state-changing endpoint.

## Key risks
- **Lax still attaches the cookie on cross-site top-level GETs.** Any mutation reachable via GET, HEAD, a 302, a `<a target=_blank>`, a client redirect, or a framework method-override is classic CSRF from email, chat, or a random webpage.
- **SameSite ≠ same-origin.** `https://evil.example.com` → `https://app.example.com` is same-site. Lax cookies go out on POST/fetch/XHR. XSS, a taken-over subdomain, a preview app, or UGC on the registrable domain fully bypasses Lax.
- **Clients lie.** Old browsers, in-app WebViews, and some native wrappers ignore or mishandle SameSite. `Secure`/`HttpOnly` do not help here.
- **New endpoints regress silently.** Dropping tokens means the next intern-added “delete?id=” link ships as an open CSRF.

## Single biggest risk
**Cross-site top-level GET (and GET-shaped) mutations still carry the session cookie.** That is Lax by design, exploitable with a link, and independent of CORS, subdomain inventory, or browser vintage. Treating Lax as “CSRF solved” makes this class of bug unauthenticated-attacker-trivial.

## Cheapest disproof
From a page you do not control (`https://evil.example`), issue a top-level GET to every mutating route (and any alias: HEAD, `?_method=`, trailing-slash, 302). **Fail the design if any state change commits.** Second check, same bar: host a form POST on a sibling subdomain of the cookie’s site; if the POST succeeds, Lax was never a complete control.
