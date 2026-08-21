# Decision C08 - is SameSite=Lax enough against CSRF

A web app sets its session cookie `SameSite=Lax; Secure; HttpOnly` and concludes
it no longer needs CSRF tokens for state-changing requests. **Question:** is
`SameSite=Lax` alone sufficient CSRF protection for all state-changing endpoints?
Commit and name the single biggest risk.
