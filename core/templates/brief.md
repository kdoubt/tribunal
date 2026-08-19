# Frozen Brief - [one-line name of the decision]

<!-- The original brief is immutable once Round 0 starts and is attached
     verbatim to every prompt in every round. Amendments are append-only,
     versioned, timestamped, and delivered identically to every seat before
     any further round. -->

## Artifact(s)

<!-- Repository-relative paths or sanitized URLs - never usernames, home
     directories, credentials, signed URLs, or private hostnames. Seats
     read these directly - list everything they need, nothing they
     shouldn't see. Invoke seats with cwd = the artifact root so relative
     paths resolve. -->

- `path/to/artifact`

## Question under review

<!-- The precise decision. If there are sub-questions, number them (Q1, Q2). -->

## Decision criteria (owner-supplied)

<!-- What a good answer optimizes for. If you want the orchestrator to break
     ties mechanically (pre-delegation), state the tie-break criteria here
     explicitly - otherwise unresolved disputes go to you. -->

-

## Constraints

<!-- Anything that bounds acceptable answers: budget, deadline, team size,
     irreversibility, compliance. -->

-

## Output contract for seats

Maximum 6 claims (replace the number before freezing if needed), each as:

```
CLAIM: <one sentence>
EVIDENCE: <file:line, verbatim quote, or named symbol from the artifact;
          a provenance-described capture (screenshot/trace: what, when,
          how); ASSUMPTION; SPECULATIVE (unreadable dependency - name it);
          or EXTERNAL with source>
CONFIDENCE: <high | med | low>
FALSIFIER: <what concrete observation would prove this claim wrong>
```

Plus: **VERDICT INPUT** - your one-line recommendation per question.

Maximum 800 words total (replace before freezing if needed).
