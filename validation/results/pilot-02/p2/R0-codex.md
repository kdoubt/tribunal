CLAIM 1: Choose A because the pain is already cross-cutting and operational, not hypothetical.  
EVIDENCE: Kubernetes NetworkPolicy does not provide workload identity or encryption. A mesh supplies service identity, certificate rotation, and east-west mTLS without every team implementing TLS correctly.  
CONFIDENCE: 0.90  
FALSIFIER: All services already share one runtime/framework and can demonstrate automated certificate rotation plus enforced mTLS through a single maintained library.

CLAIM 2: The proposed B stack does not uniformly solve the stated problem.  
EVIDENCE: An API gateway governs north-south traffic, not ordinary service-to-service calls. OpenTelemetry standardizes telemetry but does not enforce timeouts, retries, or encryption. Shared libraries drift by language, version, and team adoption.  
CONFIDENCE: 0.92  
FALSIFIER: CI and admission controls can prove every east-west client uses current security and resilience libraries, with no unsupported protocols or runtimes.

CLAIM 3: Fifteen services is enough to justify a mesh when failures already arise at service boundaries.  
EVIDENCE: The relevant threshold is coordination cost, not service count. Uniform infrastructure policy removes approximately 15 separate implementations now and continues covering new services automatically.  
CONFIDENCE: 0.82  
FALSIFIER: An inventory shows fewer than three meaningful east-west paths or essentially no cross-team/runtime variation.

CLAIM 4: Prefer a deliberately narrow deployment—especially Linkerd unless Istio-specific routing or extensibility is required.  
EVIDENCE: Starting with identity, mTLS, golden metrics, and explicit timeout policy limits configuration surface. Retries should initially remain off or tightly budgeted because mesh-level retries can amplify overload and duplicate non-idempotent operations.  
CONFIDENCE: 0.78  
FALSIFIER: The required protocols, authorization model, or traffic controls are unsupported by the selected mesh, forcing extensive custom components.

VERDICT: **A — adopt a service mesh now.**

Single biggest risk: the platform team turns the mesh into a complex traffic-programming platform, creating an opaque new failure domain and more toil than it removes.

Cheapest discriminating test: mesh two representative services and one dependency for two weeks. Measure platform/operator hours, certificate or connectivity incidents, telemetry completeness, and p95 latency. The choice is wrong if the pilot cannot eliminate manual mTLS handling and tracing gaps while adding less than roughly 2 ms p95 latency and no recurring specialist intervention.
