---
name: tribunal
description: >
  Convene an adversarial review panel of heterogeneous frontier CLIs (e.g.
  Codex CLI + Grok CLI) that debate each other through structured,
  orchestrator-relayed rounds, producing an adjudicated verdict that
  preserves dissent. Use for high-stakes, ambiguous, or irreversible
  decisions. Not for anything a test, compiler, or grep can settle.
---

# Tribunal - Claude Code adapter

**Status: maintained recipe (not a runner). The orchestrating model fills
the ledger; the human reads the verdict. Verified 2026-08. No SLA.**

Claude Code acts as the **orchestrator** defined in `core/CONTRACT.md`. This
file contains only Claude-specific mechanics; the methodology itself lives
in the tribunal repository's `core/` - read `METHODOLOGY.md`,
`CONTRACT.md`, `LEDGER.md`, and `VERDICT.md` before your first panel, and
follow them exactly.

## Install

```bash
# clone the repo (anywhere - ~/tribunal is just an example), then
# install this skill:
git clone https://github.com/kdoubt/tribunal.git ~/tribunal
mkdir -p ~/.claude/skills/tribunal        # user-scoped; or .claude/skills/ in a project
cp ~/tribunal/adapters/claude-code/SKILL.md ~/.claude/skills/tribunal/SKILL.md
export TRIBUNAL_ROOT=~/tribunal      # persist in your shell profile
```

Every `core/` reference in this skill resolves as
`$TRIBUNAL_ROOT/core/…`. Do not commit a machine-specific path into
the skill file. Installation is additive - skills are one folder each;
nothing pre-existing is touched.

## Orchestration mechanics (Claude Code specifics)

- **Spawn seats with the Bash tool in background mode**, one call per seat,
  issued in the same message so they run in parallel. Collect outputs on
  the completion notifications. Never run seats sequentially with their
  outputs in your context between spawns - that violates CONTRACT
  obligation 7 (isolation).
- **Write every prompt to a file first** (scratchpad), then invoke the seat
  with the file's contents. Use quoted heredocs (`<<'EOF'`) when
  generating prompt files.
- **Seat commands:** any genuinely heterogeneous pair - see the dated
  examples block in `adapters/shell/README.md` (the single home for
  vendor-current flags). Always wrap in `timeout` and check exit codes.
- **Smoke-test both seats** with a small verifiable question (arithmetic -
  not "reply OK") before Round 0.
- **Scan every seat output before ledgering** for the three silent seat
  killers (narration-only permission death, usage/quota-limit messages,
  truncation) - see "Silent seat killers" in `adapters/shell/README.md`.
  Exit 0 + non-empty ≠ a position. Re-run a dead seat on the SAME vendor.
- **Keep the ledger as a file** you edit between rounds - copy
  `$TRIBUNAL_ROOT/core/templates/ledger.md`. Your conversation
  context is NOT the ledger.
- **Verbatim relay:** build Round 1 inputs by concatenating the r1
  template + frozen brief + the seat's OWN Round 0 claims + the other
  seats' disputed claims, all extracted verbatim from the round files
  (`sed -n` line ranges after reading the file - then verify the
  extraction by reading the assembled prompt before sending).
- **Your three hats:** when relaying, do not summarize or comment; when
  verifying, use your file tools to check every cited `file:line` before
  relay (checked-and-failed → `dropped`; couldn't-check → relay stamped
  `UNVERIFIED`); when adjudicating, fill
  `$TRIBUNAL_ROOT/core/templates/verdict.md` from the ledger only -
  no arguments the seats didn't make.
- **Report the verdict to the user** with surviving dissent intact, and
  your adjudication criteria stated wherever you applied pre-delegated
  tie-breaks.

## Seat authentication

Each seat CLI needs its vendor's one-time authentication (typically an
OAuth device flow, per user per machine) - consult that vendor's current
docs. One durable gotcha (as of 2026-08): environment API keys can take
precedence over subscription auth and bill per token - see the note in
`adapters/shell/README.md`.
