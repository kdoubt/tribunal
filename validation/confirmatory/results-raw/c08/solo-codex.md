## Decision memo — C08

**CALL:** No. Keep CSRF tokens on every state-changing endpoint; treat `SameSite=Lax` only as defense in depth.

**Key risks:**

- `Lax` still sends the session cookie during cross-site, top-level navigations using “safe” methods. Any state-changing `GET`, `HEAD`, or method-override path remains forgeable. This behavior is specified by [RFC6265bis](https://datatracker.ietf.org/doc/draft-ietf-httpbis-rfc6265bis/).
- SameSite compares **sites, not origins**. A compromised, malicious, user-hosted, or dangling sibling subdomain is same-site and can submit authenticated `POST` requests to the app.
- Legacy or embedded clients may not enforce SameSite consistently.
- Client-side CSRF—attacker-controlled input causing trusted application JavaScript to send a request—is outside SameSite’s protection.
- `Secure` prevents cleartext cookie transport, and `HttpOnly` prevents JavaScript from reading the cookie; neither authenticates the intent or origin of a request.

**Single biggest risk:** A malicious or compromised sibling subdomain bypasses `SameSite=Lax` across the entire state-changing API because its requests are same-site. This is systemic, easily overlooked, and not limited to accidental state-changing `GET`s.

**Cheapest falsification test:** Host a page on an untrusted sibling subdomain—or a local equivalent using two origins under one registrable domain—and submit forms to representative `POST`, `PUT`-override, and destructive endpoints. Inspect the request in a current browser. **The team chose wrong if any state change succeeds with the session cookie but without a valid CSRF token.**

Adopt the framework’s synchronizer-token protection (or a correctly signed double-submit scheme) and retain `SameSite=Lax; Secure; HttpOnly`. Also reject state changes over safe methods and consider Origin/Fetch-Metadata validation. OWASP explicitly says SameSite generally does not replace proper CSRF protection. [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
