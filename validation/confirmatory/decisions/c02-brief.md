# Decision C02 - REST vs gRPC for a new internal service mesh boundary

A platform team is standardizing the transport for **internal service-to-service
calls** across ~25 services (polyglot: Go, Python, Node). Today it is ad-hoc
JSON/HTTP. They must pick one standard:

- **A) REST/JSON over HTTP** (+ OpenAPI schemas, existing tooling/observability).
- **B) gRPC** (+ protobuf schemas, codegen, HTTP/2 streaming, stricter contracts).

Latency is not currently a bottleneck; the pain is inconsistent contracts,
breaking changes, and polyglot client drift. **Commit to A or B**, name the
single biggest risk of your choice, and give the cheapest test or criterion that
would tell the team they chose wrong. (No sealed truth: this is a genuinely
contested call, judge-scored on usefulness, not accuracy.)
