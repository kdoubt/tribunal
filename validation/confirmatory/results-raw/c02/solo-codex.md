DECISION MEMO — C02

CALL

B

Standardize on gRPC with protobuf contracts and generated clients.

The stated problem is contract inconsistency, breaking changes, and polyglot client drift—not latency. gRPC addresses that problem at the boundary: one schema becomes the source of truth, generated Go/Python/Node clients reduce hand-maintained interpretation, and protobuf evolution rules make compatibility enforceable in CI.

Tradeoffs and risks:

- Higher operational and developer complexity: code generation, HTTP/2-aware infrastructure, debugging tools, and language-specific runtime behavior.
- Browser access and ad hoc inspection are worse than REST/JSON.
- Protobuf does not prevent bad evolution by itself; enforce compatibility checks, reserve removed fields, and ban unreviewed breaking changes.
- Streaming adds conceptual and reliability complexity; use unary RPCs by default and require justification for streaming.
- Generated clients can create tightly coupled release workflows unless schemas and compatibility policy are managed independently.

Single biggest risk: the organization fails to make protobuf generation and compatibility checks fast and boring, so teams bypass the standard or pin stale generated clients—recreating client drift with more machinery.

Cheapest falsification test: pilot gRPC on three representative services—one each owned primarily in Go, Python, and Node—for four weeks. Require a non-breaking schema change that all three consumers adopt independently. The choice is wrong if a developer cannot clone, generate, test, and call the service within 30 minutes using documented tooling, or if the schema change requires coordinated producer/consumer deployment.
