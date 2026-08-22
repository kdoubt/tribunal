DECISION MEMO — C02: Internal S2S transport

B

The stated failure mode is contract entropy, not latency. Protobuf + codegen makes the schema the compile artifact, not a wiki page. Buf (or equivalent) breaking-change CI, generated stubs in Go/Python/Node, and a single RPC surface kill the three pains they named: inconsistent contracts, silent JSON field drift, and hand-rolled polyglot clients. REST/OpenAPI can describe the same thing; it will not enforce it. They already failed at HTTP/JSON discipline. Keeping A is betting that more OpenAPI process will succeed where ad-hoc JSON did not.

Tradeoffs
- A preserves curl, logs, existing HTTP mesh/observability, and zero wire-format migration. It also preserves the current failure mode unless they add generated clients + consumer contract tests they do not have today.
- B buys mechanical compatibility and typed clients. It costs HTTP/2 mesh config, binary payloads, weaker Node DX, and a 25-service cutover. Streaming is unused capacity; do not sell it as the reason.
- Do not mix: one standard. Transcoding at the edge is allowed; dual internal stacks is not.

Biggest risk
Node (and secondarily Python) runtime/tooling tax plus undebuggable binaries. Teams will smuggle JSON over a “temporary” HTTP path, freeze migrations, or fork stubs — recreating client drift inside a gRPC costume.

Wrongness test (cheap)
Stand up one real unary RPC, generate clients for all three languages, and run a staged failure (deadline, unknown field, non-idempotent retry) through the existing mesh and observability stack.

Fail the choice if any of: (1) Node or Python cannot produce a working generated client and inspect a failed call in <1 engineering day without new vendor tooling; (2) after the first 5 services, ≥20% of internal call volume is still ad-hoc JSON/HTTP; (3) a breaking proto change is not rejected in CI before merge. Any one is sufficient to revert to A with mandatory OpenAPI-first + generated clients.
