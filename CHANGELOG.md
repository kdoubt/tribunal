# Changelog

## 0.1.0 — 2026-08-19

Initial public extraction. These documents were reviewed in-repo by a
10-seat cross-vendor panel (4 Codex CLI seats, 4 Grok CLI seats, 2 Claude
seats — developer + product roles with orchestrator-relayed cross-talk);
the methodology itself still has one published run (see README, Status).

- `core/`: methodology, contract, ledger, verdict — extracted
  orchestrator-neutral from the original Claude Code skill.
- `core/templates/`: scout (adopter onboarding), brief, r0-seat, r1-seat,
  r2-revision, ledger, verdict.
- `adapters/claude-code/`: the original skill, thinned to host mechanics
  (`PRESSURE_TEST_ROOT` indirection; vendor flags live in the shell
  adapter's dated examples block).
- `adapters/shell/`: portable scripted recipe for any orchestrator (GNU
  timeout detection, wait-both-fail-loud, smoke test, mechanical Round 1
  assembly including each seat's own Round 0 claims).
- `adapters/{codex-cli,gemini-cli,opencode}/`: marked stubs (untested).
- `examples/sample-run/`: the real two-seat panel that designed the
  methodology, labeled as a historical bootstrap transcript with its
  known non-compliances enumerated.
