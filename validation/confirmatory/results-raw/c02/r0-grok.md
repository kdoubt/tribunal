**CLAIM 1:** The named pain is schema enforcement and client generation, not HTTP vs HTTP/2. Protobuf + stubs + breaking-change CI is harder to bypass than OpenAPI-on-JSON.
**EVIDENCE:** JSON objects accept extra/missing fields at runtime; generated protobuf clients fail at compile/codegen time. Buf-style breaking checks are routine; OpenAPI codegen is optional and often skipped, which is how ad-hoc JSON/HTTP got here.
**CONFIDENCE:** 0.82
**FALSIFIER:** A two-week OpenAPI + generated clients + oasdiff/spectral CI spike matches protobuf’s catch rate for type/field breaks, with no hand-written JSON clients remaining.

**CLAIM 2:** Go/Python/Node can speak gRPC, but Python and Node are the weak seats and will drive escape hatches.
**EVIDENCE:** grpc-go is native; Python `grpcio` and Node `@grpc/grpc-js` work, yet version skew, async, deadlines, and error mapping are the usual stall. Latency is unused, so the only gRPC win that matters is contract rigidity.
**CONFIDENCE:** 0.76
**FALSIFIER:** Python and Node ship a Go↔Python and Node↔Go RPC (deadlines, status, one schema change) in comparable time to REST, with no request for a JSON fallback.

**CLAIM 3:** Existing REST observability is a real on-call tax for B, not a reason to keep A’s failed contract model.
**EVIDENCE:** curl, JSON logs, and body-capturing APM work today. gRPC needs proto-aware traces, `grpcurl`, and a non-HTTP error model. Mesh HTTP/1.1 or JSON-payload assumptions will bite during rollout.
**CONFIDENCE:** 0.84
**FALSIFIER:** On-call debugs a failed RPC from existing mesh/logs as fast as a failed REST call, without a sidecar JSON gateway.

**CLAIM 4:** A without mandatory codegen + breaking-change gates reproduces today’s drift; B’s risk is a split stack, not worse JSON.
**EVIDENCE:** The fleet already has HTTP/JSON and still has inconsistent contracts. A second unofficial JSON path next to gRPC is the common failure, not protobuf itself.
**CONFIDENCE:** 0.78
**FALSIFIER:** Current services already generate clients from OpenAPI and still drift the same way protobuf would.

**VERDICT: B** — Biggest risk: Python/Node/ops friction creates shadow JSON APIs and two standards. Cheapest test: one Go↔Python and one Node↔Go RPC with protobuf, buf breaking checks, and a live failed-call debug; if either team needs a JSON escape hatch or debug time is >2× REST, switch to A with mandatory OpenAPI codegen + breaking-change CI.
