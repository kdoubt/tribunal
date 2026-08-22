## Decision memo — C05

**CALL:** Reject this configuration; add a startup probe sized beyond worst-case warm-up, a readiness probe to gate traffic, and reserve liveness for unrecoverable hangs.

**Key risks:**

- Without a readiness probe, Kubernetes treats the container as ready by default, so the Service may route traffic during its ~40-second warm-up.
- With default `periodSeconds: 10`, liveness begins after 10 seconds and can reach three consecutive failures around 30 seconds—restarting the JVM before it becomes healthy and potentially creating an endless restart loop.
- A transient `/health` stall is interpreted as a reason to restart, even when temporarily removing the Pod from traffic would be safer.
- The default one-second probe timeout can amplify failures during JVM pauses or resource contention.

Kubernetes explicitly assigns these roles: readiness controls traffic, liveness triggers restarts, and startup suppresses the other probes until initialization succeeds. [Kubernetes probe documentation](https://kubernetes.io/docs/concepts/workloads/pods/probes/)

**Single biggest risk:** A self-sustaining crash loop in which liveness repeatedly kills the JVM before its 40-second warm-up completes, leaving the deployment with no usable capacity.

**Cheapest decisive test:** Deploy one replica with a deterministic 40-second startup delay and the proposed probe settings; watch Pod restart count and Service endpoints. The team chose wrong if the Pod restarts before first becoming healthy **or** appears as a Service endpoint before it can serve a real request. This configuration is expected to fail both criteria.
