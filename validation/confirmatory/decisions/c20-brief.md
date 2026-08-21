# Decision C20 - comparing a request's HMAC signature

A webhook receiver recomputes the expected HMAC-SHA256 signature and compares it
to the one in the request header with a normal string equality check
(`if received == expected`). **Question:** is ordinary `==` comparison of the two
hex signature strings adequate here? Commit and name the single biggest risk.
