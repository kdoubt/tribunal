**CALL:** Reject. This will not behave correctly on slow starts or transient stalls. Use a startup probe (or long initialDelay) plus a readiness probe; keep liveness conservative.

**Why it fails**

Liveness does not gate traffic. With no readiness probe, the pod is Ready as soon as the container starts, so Kubernetes sends traffic through the entire ~40s JVM warmup.

Liveness *does* kill the container. Defaults are `periodSeconds=10`, `timeoutSeconds=1`. First check at t=10s; three failures ⇒ restart around t=30s — before warmup finishes. Result: CrashLoopBackOff on every deploy. A restart also throws away JIT/heap warmup, so a stall that should have been brief becomes a cold start.

Transient stalls (GC, thread-pool saturation, lock) fail `/health` (often on the 1s timeout) and trigger a restart instead of shedding load. Readiness would have dropped endpoints; liveness reboots the patient.

**Key risks**

- Startup death spiral: never reaches a stable Running/Ready state.
- Traffic to a cold, not-listening JVM → 5xx/connection errors for ~40s (or forever if it loops).
- Stall-induced restart storms under load; lost warmup; longer outage than the stall.
- `/health` conflates “process alive” with “safe to serve,” so one endpoint cannot serve both probes well.

**Single biggest risk**

Kubelet restarts the JVM during warmup. The service never becomes healthy; deploys and node reschedules fail closed in a restart loop.

**Cheapest disproof**

Deploy one pod; watch 90s (`kubectl get pod -w`, restart count, Ready, endpoints).

You chose wrong if **either**:

1. `restartCount` increments before ~40s of continuous Ready, or
2. the pod is Ready/in endpoints while `/health` is still failing or the process is still warming.

(1) is the expected outcome of this config.
