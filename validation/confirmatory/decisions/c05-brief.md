# Decision C05 - health probe for a slow-starting service

A JVM service takes ~40s to warm up. To keep Kubernetes from sending traffic
before it is ready, a dev adds a **liveness** probe hitting `/health` with a 10s
initialDelay and 3-failure threshold, and no readiness probe. **Question:** will
this configuration behave correctly during slow starts and transient stalls?
Commit and name the single biggest risk.
