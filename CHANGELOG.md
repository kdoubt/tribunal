# Changelog

## 0.2.0 - 2026-08-20

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

## 0.1.0 - 2026-08-19

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
