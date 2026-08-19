# Contributing

This is a methodology repository with a solo maintainer. All contributions
must be MIT-licensable (see `LICENSE`). The support policy is deliberately
narrow:

- **Complete adapters** exist only for orchestrators the maintainer actually
  runs (each adapter's README states its own status).
- **Stub adapters** are invitations, not promises: a stub directory means
  "this orchestrator should be able to implement `core/CONTRACT.md`
  (untested); nobody has written it up yet." Stubs carry no SLA and no
  compatibility claim.
- **Contributed adapters** are community-maintained and experimental. They
  stay attributed to their contributors, who are expected to keep them
  current; an adapter without an active maintainer or a current sanitized
  run is demoted back to stub status (its run transcript archived) rather
  than maintained by the project. An adapter PR must:
  1. translate host mechanics only - it MUST NOT redefine round counts,
     ledger states, grounding obligations, or verdict rules (see
     `core/CONTRACT.md`);
  2. include one real sanitized run (brief → R0 → R1 → ledger → verdict)
     demonstrating the adapter end to end;
  3. contain no personal data, secrets, or environment-specific paths.

**Most-wanted contribution:** a sanitized panel run on a conventional
engineering decision (architecture call, migration, security boundary) -
the current sample run is the methodology reviewing itself, and an
independent real-world transcript is the strongest evidence the repo can
gain.

## Issues / PRs

`core/` factual bugs and sanitized-run PRs only. Adapter CLI flag breakage
→ that vendor's docs (flags are date-stamped examples, not supported
surface). Stub directories are invitations, not tickets. No response SLA.
An adapter without an active contributor is reverted to stub in-tree.

## Changes to `core/`

`core/` is normative. Changes to rounds, ledger semantics, grounding, or
verdict rules need strong evidence - ideally a documented panel run where
the current rule produced a bad outcome. Editorial fixes are always
welcome. Normative edits increment the CHANGELOG version (minor bumps
until 1.0); adapter PRs must cite the version they were verified against.

## What will be declined

- Runners, frameworks, SDKs, plugin systems. The method's portability comes
  from being documents; code would become the product and rot.
- Vendor-specific marketing, benchmarks, or "which model is better"
  content. Seats are deliberately interchangeable.
- Adapter matrices that imply support the maintainer cannot provide.
