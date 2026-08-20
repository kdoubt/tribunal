# Changelog

All notable changes are recorded here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows
[Semantic Versioning](https://semver.org/spec/v2.0.0.html) as scoped for a
methodology repo in [`CONTRIBUTING.md`](CONTRIBUTING.md#versioning). Changes
accumulate under **Unreleased** and are stamped with a version and date when
a release is cut; published tags are immutable.

## [Unreleased]

## [0.3.4] - 2026-08-20

- **Versioning policy** (`CONTRIBUTING.md`): SemVer scoped for a docs/method
  repo - MINOR = a normative `core/` change that could alter a panel's
  conduct or outcome; PATCH = clarifications, doc/tooling fixes, adapters,
  examples; MAJOR reserved for 1.0 and later breaking normative changes.
  Records the Keep-a-Changelog release flow and that published tags are
  never moved.

## [0.3.3] - 2026-08-20

- **Docs consistency pass.** README "Status" was stale at v0.1.0; rather than
  hardcode a version that re-staled on every release, the line now points at
  the dynamic release badge and `CHANGELOG.md` - no version string to drift.
  `CHANGELOG.md` added to the repository-layout listing (the update notice
  points users there). Repo-layout `scout` line notes the weekly update
  notice.
- **`brief.md` gains an optional "Lens assignments" section** so the
  exclusive-lens mechanism from the v0.3.0 doctrine is usable end-to-end
  from the template - default-empty, with inline guidance to prefer additive
  lenses in Decision criteria and reserve the matrix for large panels with
  >=2 heterogeneous seats per surface plus a retained generalist.

## [0.3.2] - 2026-08-20

- **`scout` update check now handles zip/tarball installs too.** A copy
  with no `.git` has no `origin` to pull from, so the git check no-oped for
  those users. Now `scout` detects a non-git copy, reads its version from
  `CHANGELOG.md`, and (throttled once per 7 days via a `$XDG_CACHE_HOME`
  timestamp) points the user at the git-clone install that self-checks plus
  the releases page. Same guarantees as the git path: stderr-only,
  network-free, fail-silent, notify-only, `TRIBUNAL_NO_UPDATE_CHECK=1` to
  opt out. The git-clone path stays write-free (throttled by `.git/FETCH_HEAD`
  mtime); only the zip fallback writes a small cache timestamp.
- **README "Staying current"**: documents the zip/tarball case and why the
  git-clone install is recommended (every future update becomes `git pull`).

## [0.3.1] - 2026-08-20

- **`scout` weekly update check** (notify-only, never auto-pulls). At most
  once per 7 days (throttled by `.git/FETCH_HEAD` mtime), if the clone is
  behind origin, `scout` prints a one-line update notice with the exact
  `git ... pull --ff-only` command. The notice goes to **stderr only**, so
  it can never contaminate the prompt piped to your agent on stdout;
  fail-silent on no-git/no-network/detached-HEAD; opt out with
  `TRIBUNAL_NO_UPDATE_CHECK=1`. No runtime added to the repo - this stays a
  docs-only clone; the check just surfaces that a `git pull` is available.
- **README "Staying current"**: documents the update command, the weekly
  notice, and why updating is non-destructive - `--ff-only` is clean-or-
  abort (untracked files untouched, local edits never silently overwritten)
  - plus the keep-the-clone-pristine convention that guarantees it.

## [0.3.0] - 2026-08-20

- **Lens-assignment doctrine** (`core/METHODOLOGY.md`, "Assigning lenses"):
  codifies WHEN a panel should give seats distinct review lenses vs. run
  raw heterogeneous seats. Two mechanisms - **additive** (surfaces named in
  the shared brief, legal at any N, the default) vs. **exclusive slice** (a
  seat prioritizes one surface via a labeled addendum, the one sanctioned
  departure from the identical-brief rule). Exclusive slicing is fenced:
  ≥2 heterogeneous seats per lens, capped at floor(N/2), a retained
  generalist read, and Round 1 still relays every disputed claim to every
  seat. `core/CONTRACT.md` obligation 1 gains a scoped, explicit exception
  so the exclusive `LENS:` addendum is sanctioned by the MUST layer instead
  of silently contradicting "delivered identically to every seat" - the
  addendum is part of the frozen brief, fixed before Round 0, with every
  seat receiving the full assignment matrix and identical criteria. Whether
  a surface earns a lens is judgment (own criterion,
  non-implied, salient); the staffing floor is a MUST. `scout` step 4 now
  defaults first panels to raw seats and only recommends exclusive slicing
  at N ≥ 4 with proper staffing. This section was itself pressure-tested by
  a 2-seat Codex+Grok panel, which caught the identical-brief conflict and
  the unsupported "~5 seats" gate the first draft shipped with.

## [0.2.0] - 2026-08-20

- **Renamed the project: Pressure-Test → Tribunal** (the old GitHub URL
  redirects). Skill name, `TRIBUNAL_ROOT`, and install paths renamed to
  match; the historical sample-run transcripts are unmodified as always.
- **`scout` helper** (repo root): one runnable command feeds the scouting
  prompt to your agent (resolves the clone-path placeholder, strips
  template chrome). Text-only — runs no panel, writes nothing.
- **README**: featured one-command clone + scout up top; hero workflow
  diagram (light/dark SVG) now shows scout and the triggered Round 2, so
  the chart matches the full template set; FAQ targeting real search
  queries; honest getting-started framing (setup is seconds; the 20-40 min
  is the panel itself); documented that seats scale to N ≥ 2.
- **Adapters**: silent-seat-killer taxonomy (permission death, quota
  exhaustion, truncation); local models documented as first-class seats.
- Docs: em/en-dashes replaced with plain hyphens (verbatim transcripts
  preserved); confident Status rewrite; LICENSE holder set with website.

## [0.1.0] - 2026-08-19

Initial public extraction. These documents were reviewed in-repo by a
10-seat cross-vendor panel (4 Codex CLI seats, 4 Grok CLI seats, 2 Claude
seats - developer + product roles with orchestrator-relayed cross-talk);
the methodology itself still has one published run (see README, Status).

- `core/`: methodology, contract, ledger, verdict - extracted
  orchestrator-neutral from the original Claude Code skill.
- `core/templates/`: scout (adopter onboarding), brief, r0-seat, r1-seat,
  r2-revision, ledger, verdict.
- `adapters/claude-code/`: the original skill, thinned to host mechanics
  (`TRIBUNAL_ROOT` indirection; vendor flags live in the shell
  adapter's dated examples block).
- `adapters/shell/`: portable scripted recipe for any orchestrator (GNU
  timeout detection, wait-both-fail-loud, smoke test, mechanical Round 1
  assembly including each seat's own Round 0 claims).
- `adapters/{codex-cli,gemini-cli,opencode}/`: marked stubs (untested).
- `examples/sample-run/`: the real two-seat panel that designed the
  methodology, labeled as a historical bootstrap transcript with its
  known non-compliances enumerated.

[Unreleased]: https://github.com/kdoubt/tribunal/compare/v0.3.4...HEAD
[0.3.4]: https://github.com/kdoubt/tribunal/compare/v0.3.3...v0.3.4
[0.3.3]: https://github.com/kdoubt/tribunal/compare/v0.3.2...v0.3.3
[0.3.2]: https://github.com/kdoubt/tribunal/compare/v0.3.1...v0.3.2
[0.3.1]: https://github.com/kdoubt/tribunal/compare/v0.3.0...v0.3.1
[0.3.0]: https://github.com/kdoubt/tribunal/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/kdoubt/tribunal/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/kdoubt/tribunal/releases/tag/v0.1.0
