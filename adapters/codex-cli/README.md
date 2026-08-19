# Codex CLI as orchestrator — STUB

**Status: stub. No implementation yet, no support commitment.**

Codex CLI should be able to orchestrate a Pressure-Test panel (untested): it likely satisfies the
orchestrator role in `core/CONTRACT.md` (can run shell commands, read
files, and maintain a ledger file). Seats would then be other vendors'
CLIs (e.g. Claude Code as a seat via `claude -p`, plus Grok CLI).

To implement: follow `adapters/shell/README.md` mechanics driven from a
Codex session, honoring every MUST in `core/CONTRACT.md` — especially
parallel isolated Round 0 and verbatim relay.

Contributions welcome — see `CONTRIBUTING.md` for the adapter policy.
