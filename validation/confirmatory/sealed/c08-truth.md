# C08 SEALED
- correct_call: **NOT sufficient (defense-in-depth, not a full replacement).**
  SameSite=Lax still allows **top-level GET** cross-site navigations to send the
  cookie, so any state change reachable via GET (or same-site subdomains, or
  clients/browsers that don't enforce SameSite) is exposed; Lax is a strong
  mitigation but OWASP still recommends CSRF tokens for state-changing requests.
- oracle: OWASP CSRF Prevention Cheat Sheet; MDN SameSite.
- fix: keep anti-CSRF tokens (or double-submit) for state-changing ops; never
  perform state changes on GET; treat SameSite as defense-in-depth.
- must_catch: [Lax still sends cookie on top-level GET; state changes must not be on GET; SameSite is defense-in-depth not a full CSRF replacement; subdomain/legacy-browser gaps]
- landmine: "sufficient, SameSite=Lax stops CSRF" as a blanket claim.
