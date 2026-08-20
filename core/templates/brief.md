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
     paths resolve.
     If the decision's correctness depends on rules defined elsewhere - a
     contract, schema, spec, style guide, or (for a change to this repo) its
     own CONTRACT/LEDGER - paste that governing text in as an artifact or
     inline it here. A seat checking against pasted text catches consistency
     collisions that a seat reconstructing the rule from memory misses. -->

- `path/to/artifact`

## Question under review

<!-- The precise decision. If there are sub-questions, number them (Q1, Q2). -->

## Decision criteria (owner-supplied)

<!-- What a good answer optimizes for. If you want the orchestrator to break
     ties mechanically (pre-delegation), state the tie-break criteria here
     explicitly - otherwise unresolved disputes go to you. -->

-

## Lens assignments (optional)

<!-- Leave this out for almost every panel. By default all seats get this
     identical brief; if the artifact has distinct review surfaces, name
     them as additive lenses in Decision criteria above ("every seat also
     addresses security / DX"), NOT here.
     Fill this in ONLY for an exclusive slice (see METHODOLOGY, "Assigning
     lenses"): a large panel where each material surface has >=2
     heterogeneous seats. List the full matrix - every seat sees all of it,
     and each is told to PRIORITIZE its surface without excluding
     decision-relevant findings elsewhere. Keep at least one generalist seat
     with no LENS line. -->

<!-- LENS <seat/vendor>: prioritize <surface> (e.g. "seat A (codex): security") -->

## Constraints

<!-- Anything that bounds acceptable answers: budget, deadline, team size,
     irreversibility, compliance. -->

-

## Output contract for seats

Maximum 6 claims (replace the number before freezing if needed), each as:

```
CLAIM: <one sentence>
EVIDENCE: <a DECISIVE pointer - the smallest file:line, verbatim span, named
          symbol, or oracle invocation that could settle the claim, not an
          argument; a provenance-described capture (screenshot/trace: what,
          when, how); ASSUMPTION; SPECULATIVE (unreadable dependency - name
          it); or EXTERNAL with source>
CONFIDENCE: <a 0-1 probability (preferred - it is scored against oracle
          outcomes later, so calibrate); or high | med | low, scored as
          0.85 | 0.6 | 0.3>
FALSIFIER: <what concrete observation would prove this claim wrong>
```

Plus: **VERDICT INPUT** - your one-line recommendation per question.

Maximum 800 words total (replace before freezing if needed).
