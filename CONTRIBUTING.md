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
welcome. Adapter PRs must cite the version they were verified against.

## Versioning

Tribunal follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html),
scoped for a methodology repo (there is no code API to break, so the
"contract" is what a panel is *required* to do):

- **MAJOR** (`1.0.0`+) - a breaking change to a normative `core/` rule after
  1.0. Pre-1.0 the project is explicitly unstable; anything may change under
  a MINOR bump. `1.0.0` is the first stability commitment, cut once the core
  has held across independent real-world runs.
- **MINOR** (`0.x.0`) - a normative change: a rule in `core/` added, removed,
  or altered such that a panel run could be *conducted or adjudicated
  differently* (e.g. the lens-assignment doctrine). New maintained adapters
  also land here.
- **PATCH** (`0.x.y`) - anything that does not change what the method
  requires: clarifications, doc/README fixes, `scout` and other tooling,
  new stubs or examples, typo and formatting passes.

When unsure whether an edit is normative, ask: *could a seat or orchestrator
following the old text reach a different verdict than one following the new
text?* If yes, it is MINOR; if no, PATCH.

The [`CHANGELOG.md`](CHANGELOG.md) is the source of record and follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/):

1. Every change is written under `## [Unreleased]` as part of its PR -
   recording a change is decoupled from cutting a release, so several
   changes can batch into one version.
2. A release renames `[Unreleased]` to `## [x.y.z] - YYYY-MM-DD`, adds a
   fresh empty `[Unreleased]`, updates the compare-link footer, then tags
   `vx.y.z` with an annotated tag (`git tag -a`).
3. **Published tags are immutable.** A shipped tag is never moved or
   force-pushed; if a release was wrong, fix forward with the next PATCH.
   The git tag is the single source of truth for "current version" - the
   README shows it via a dynamic badge, and nothing hardcodes a version
   string in prose.

## What will be declined

- Runners, frameworks, SDKs, plugin systems. The method's portability comes
  from being documents; code would become the product and rot. The only
  scripts that belong here are small, optional, local helpers that print to
  stdout and nothing else (`scout`, `flywheel-export`) - no network, no
  backend, no service. That line is the boundary, not an invitation.
- Vendor-specific marketing, benchmarks, or "which model is better"
  content. Seats are deliberately interchangeable.
- Adapter matrices that imply support the maintainer cannot provide.
