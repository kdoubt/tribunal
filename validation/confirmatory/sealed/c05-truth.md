# C05 SEALED
- correct_call: **NOT correct.** A liveness probe that fails during the 40s warmup
  makes the kubelet **kill and restart** the container -> crash-loop that never
  finishes starting. Liveness ≠ "ready for traffic". You need a **readiness**
  probe to gate traffic, and liveness must tolerate startup (startupProbe or a
  generous initialDelay/threshold).
- oracle: Kubernetes docs on liveness/readiness/startup probes.
- fix: readiness probe to gate Service traffic; startupProbe (or large
  failureThreshold*period) so liveness doesn't kill during warmup.
- must_catch: [liveness failure = restart (crash-loop risk); readiness is what gates traffic; startupProbe for slow starts; liveness must not trip during warmup]
- landmine: "correct, liveness keeps it healthy" / conflating liveness with readiness.
