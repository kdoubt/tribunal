# Shell Adapter - run a panel from any orchestrator (or by hand)

**Status: maintained recipe (not a runner). The human fills the ledger.
Verified 2026-08 on Linux/bash. No SLA.**

The methodology needs no framework: each round is a fresh, stateless,
single-shot CLI invocation, and the ledger - a file you maintain - is the
memory. Any orchestrator that can run shell commands (or a human with a
terminal) can run a compliant panel.

## Seat invocation - examples (verified 2026-08; flags change)

Any CLI with a non-interactive single-shot mode can be a seat. Confirm your
CLI's current headless flag with `<cli> --help` before trusting these:

```bash
codex exec -s read-only --skip-git-repo-check "<prompt>"   # Codex CLI (read-only sandbox)
grok --permission-mode plan -p "<prompt>"                  # Grok CLI (plan = read-only)
claude -p "<prompt>"                           # Claude Code (as a seat)
gemini -p "<prompt>"                           # Gemini CLI
ollama run <model> "<prompt>"                  # local model as a seat
```

Local models are first-class seats - heterogeneity is about *model
families and training lineages*, not billing. Two caveats: a local
distill of the same family as another seat is fake heterogeneity (see
CONTRACT: identity theater), and the model-selection principle still
applies - a seat must be frontier-class for a debate round, local or not.
Note that plain model runners (like `ollama run`) have no file tools, so
the orchestrator must inline the artifact excerpts the brief needs into
the prompt for that seat.

Seats must be genuinely heterogeneous - different vendors or model
families, local or hosted (CONTRACT: no identity theater). Authentication
is vendor-controlled and changes; consult each vendor's current docs. One
durable gotcha (as of 2026-08 - verify against your CLI's precedence
rules): environment API keys (`OPENAI_API_KEY`, `XAI_API_KEY`, …) can
silently take precedence over subscription auth and bill per token - keep
them unset for panel runs.

If your CLI offers a host-level read-only or sandbox mode, use it - the
prompt's read-only fence is an instruction, not a security control (see
CONTRACT "Seat fencing"). As of 2026-09 that is `codex exec -s read-only`
and `grok --permission-mode plan` (Grok also takes a `--sandbox` profile).
**Never substitute a broad shell allow-rule for a read-only mode.** A rule
like `Bash(git *)` is a shell escape, not a fence: `git` alone reaches
arbitrary command execution (`git -c core.pager=<cmd> log`,
`git -c alias.x='!<cmd>' x`) plus every mutating subcommand (`push`,
`reset --hard`, `clean -fdx`). If you must allow shell at all, allow single
read-only subcommands as separate rules (`git log`, `git show`, `git diff`).

**Silent seat killers.** Exit code 0 does not mean the seat produced a
position. Three failure shapes produce plausible-looking output and must
be caught by *reading* the output, not by exit codes:

1. **Headless permission death** - an agentic CLI hits an interactive
   tool-approval prompt, auto-cancels it, and exits 0 with only its
   opening narration ("I'll read the files…" and nothing else).
   Run seats in the CLI's read-only mode (above) so built-in read tools
   need no approval, and tell seats to prefer those tools over shell
   commands. Do not fix this with a broad shell allow-rule (see the
   `Bash(git *)` warning above). Some CLIs
   also accept the prompt from a file (e.g. Grok's `--prompt-file`),
   which avoids argv size/visibility limits.
2. **Usage/quota exhaustion mid-panel** - subscription CLIs expose no
   "remaining quota" query, so a cap hit between rounds surfaces only as
   an error message or truncated prose in the output, often with exit 0.
   The smoke test catches a seat that is *already* exhausted; for
   mid-panel hits, scan each seat file for limit signatures before
   ledgering (case-insensitive: "usage limit", "rate limit", "quota",
   "try again later", "upgrade to"). A hit is a dead seat **only when the
   required output shape is also missing** (no CLAIM block in Round 0, no
   ATTACK section in Round 1) - briefs about rate limiting, quotas, or
   upgrade paths legitimately contain these words, and a seat that used
   them while still delivering its claims is alive. A *subscription cap* is
   the sub-case with a reset time in the message ("try again at ...") and,
   typically, an empty seat file with the message on stderr - so inspect
   stderr as well. Do not substitute a vendor. Keep the round's packet
   unchanged (record its hash), do not relay the peer's answer to the capped
   seat, and re-run the same seat with the same packet once the cap lifts: a
   resumed round is the same round, and the resumed run counts as one more
   attempt, which you disclose; this does not extend the retry limit you
   set for the panel - the grounded study's fourth attempts on g04 and g08
   were disclosed deviations from its own three (`validation/grounded/results/`).
3. **Context overflow / truncation** - output that stops mid-sentence or
   omits the required sections.

All three resolve the same way, via the CONTRACT: output that is not in
the brief's required shape (CLAIM blocks / the ATTACK-CONCEDE-REVISE
structure) is **rejected and the seat re-run once** - after the cause is
fixed - and never paraphrased, summarized, or synthesized around. A dead
seat re-runs on the *same* vendor; substituting another vendor's model
sacrifices the heterogeneity the seat exists to provide.

## A complete two-seat panel

Save as a script (do not paste into an interactive shell - it uses
`exit`). Two roots, kept separate: the tribunal **clone** (templates)
and the **artifact root** - the project under review, where seats must run
so the brief's relative paths resolve:

```bash
#!/usr/bin/env bash
set -euo pipefail

TRIBUNAL_ROOT=${TRIBUNAL_ROOT:?set to your tribunal clone}
ARTIFACT_ROOT=${ARTIFACT_ROOT:?set to the project under review}
[[ -f "$TRIBUNAL_ROOT/core/templates/r0-seat.md" ]] || { echo "not a tribunal clone: $TRIBUNAL_ROOT" >&2; exit 1; }

# Seats run with cwd = ARTIFACT_ROOT so the brief's relative paths resolve.
# Panel bookkeeping (prompts, seat outputs, ledger, verdict) lives in $PANEL_OUT
# and is referenced by absolute path - do NOT `cd` into it, or the seats would
# resolve the brief's paths against panel/ instead of the project under review.
cd "$ARTIFACT_ROOT"
PANEL_OUT=${PANEL_OUT:-"$ARTIFACT_ROOT/panel"}
mkdir -p "$PANEL_OUT"

# --- Seats: name + single-shot command (see examples block above)
# Each in its CLI's read-only mode - never a broad shell allow-rule (see above).
SEAT_A_CMD=(codex exec -s read-only --skip-git-repo-check)
SEAT_B_CMD=(grok --permission-mode plan -p)

# --- GNU timeout (macOS: brew install coreutils), always with a kill-after
if   command -v timeout  >/dev/null; then T=(timeout  -k 10 600)
elif command -v gtimeout >/dev/null; then T=(gtimeout -k 10 600)
else echo "need GNU timeout (macOS: brew install coreutils)" >&2; exit 1
fi

# --- 0. Smoke-test each seat with a VERIFIABLE answer (a canned "reply OK"
#     can pass a misconfigured seat)
for s in A B; do
  cmd="SEAT_${s}_CMD[@]"
  ans=$("${T[@]}" "${!cmd}" "What is 19+23? Reply with the number only.") || { echo "seat $s dead" >&2; exit 1; }
  [[ "$ans" == *42* ]] || { echo "seat $s failed smoke test: $ans" >&2; exit 1; }
done

# --- 0b. Read-test the artifact path (diagnostic, not a gate). The smoke test
#     proves the model answers; it does not prove the seat's file tools reach
#     the artifact on THIS host. Ask each seat to quote a span you already
#     know and compare it with the source. A mismatch or a refused read means
#     something on the host intercepts reads (a read-size plugin, a sandbox
#     path rule) - fix that before Round 0, keeping the read-only fence. No
#     line threshold, no hook, nothing further once it passes. Skip with
#     SKIP_READ_TEST=1 if you have already proven the path on this host.
if [[ -z ${SKIP_READ_TEST:-} ]]; then
  READ_TEST_FILE=${READ_TEST_FILE:-README.md}      # relative to ARTIFACT_ROOT
  expect=$(grep -m1 -E '.{20,}' "$READ_TEST_FILE") || { echo "read-test: no line >=20 chars in $READ_TEST_FILE" >&2; exit 1; }
  for s in A B; do
    cmd="SEAT_${s}_CMD[@]"
    got=$("${T[@]}" "${!cmd}" "Open ./$READ_TEST_FILE and quote its first line that is at least 20 characters long, exactly as written, nothing else.") \
      || { echo "seat $s: read-test invocation failed" >&2; exit 1; }
    [[ "$got" == *"$expect"* ]] || { echo "seat $s cannot read the artifact verbatim - host read interceptor or sandbox path rule? got: $got" >&2; exit 1; }
  done
fi

# --- 1. Freeze the brief. Fill core/templates/brief.md, save as
#     $PANEL_OUT/frozen-brief.md; assemble the R0 prompt mechanically.
#     (Use quoted heredocs - <<'EOF' - if you generate prompts in-script,
#     so backticks and $() in prompt text aren't expanded.)
[[ -s "$PANEL_OUT/frozen-brief.md" ]] || { echo "write $PANEL_OUT/frozen-brief.md first (copy core/templates/brief.md)" >&2; exit 1; }
{ cat "$TRIBUNAL_ROOT/core/templates/r0-seat.md"; echo; cat "$PANEL_OUT/frozen-brief.md"; } > "$PANEL_OUT/r0-prompt.md"

# --- 2. Round 0: identical prompt, PARALLEL, no cross-exposure. Seats run from
#     ARTIFACT_ROOT (cwd) so brief paths resolve; outputs land in $PANEL_OUT.
#     These examples pass the prompt as an argv string for portability. argv is
#     visible in `ps` and bounded by ARG_MAX, so for large prompts or shared
#     hosts use your CLI's file/stdin input instead (e.g. Grok --prompt-file
#     "$PANEL_OUT/r0-prompt.md", or pipe the file on stdin where the CLI
#     supports it). The prompt file stays as the audit trail either way.
"${T[@]}" "${SEAT_A_CMD[@]}" "$(cat "$PANEL_OUT/r0-prompt.md")" > "$PANEL_OUT/seat-a-r0.md" 2> "$PANEL_OUT/seat-a-r0.err" & A_PID=$!
"${T[@]}" "${SEAT_B_CMD[@]}" "$(cat "$PANEL_OUT/r0-prompt.md")" > "$PANEL_OUT/seat-b-r0.md" 2> "$PANEL_OUT/seat-b-r0.err" & B_PID=$!
fail=0
wait "$A_PID" || { echo "seat A failed ($?)" >&2; fail=1; }
wait "$B_PID" || { echo "seat B failed ($?)" >&2; fail=1; }
(( fail == 0 )) || exit 1
[[ -s "$PANEL_OUT/seat-a-r0.md" && -s "$PANEL_OUT/seat-b-r0.md" ]] || { echo "empty seat output" >&2; exit 1; }
# limit-signature scan (see "Silent seat killers"; repeated after Round 1).
# A limit message alone is NOT a dead seat - briefs about rate limits or quotas
# legitimately contain these words - so also require the seat's output shape
# to be missing (no CLAIM block in R0; no ATTACK section in R1).
dead_seat() { # $1 = seat output file, $2 = required-shape regex, $3 = its name
  grep -qiE 'usage limit|rate limit|quota|try again later|upgrade to' "$1" || return 0
  grep -qE "$2" "$1" && return 0
  echo "$1: limit signature and no $3 - dead seat, fix and re-run it" >&2; return 1
}
for f in "$PANEL_OUT"/seat-*-r0.md; do dead_seat "$f" 'CLAIM' 'CLAIM block' || exit 1; done

# --- 3. Ledger: copy the template, fill per core/LEDGER.md, mark
#     agreed-r0 / disputed / open. If everything decision-relevant is
#     agreed-r0: STOP - write the verdict now.
cp "$TRIBUNAL_ROOT/core/templates/ledger.md" "$PANEL_OUT/ledger.md"
${EDITOR:?set EDITOR to your editor command} "$PANEL_OUT/ledger.md"

# --- 3a. Stop rule (a): if Round 0 already agreed on everything
#     decision-relevant, SKIP Round 1 - relaying non-disputed claims only
#     invites politeness convergence. BUT agreement is not verification: run the
#     oracle on every checkable agreed claim FIRST (a shared blind spot can make
#     both seats agree on a false claim), then write the verdict. Early stop
#     skips the debate, never the oracle.
read -rp "Round 0 agreed on everything decision-relevant? Skip Round 1? [y/N] " agreed
if [[ ${agreed:-} == [yY]* ]]; then
  echo "Before writing the verdict: run oracles on the checkable agreed claims" >&2
  echo "(tests/compiler/primary-doc lookups). A check that ran and FAILED means" >&2
  echo "the claim is dropped. A check that COULD NOT RUN means the claim's" >&2
  echo "status becomes disputed with the UNVERIFIED stamp (CONTRACT obligation" >&2
  echo "5) - it may NOT ride the early stop into the verdict as agreed: either" >&2
  echo "run Round 1 on it, or report it in bucket 3 (surviving dissent) with" >&2
  echo "its discriminating test - never as consensus." >&2
  cp "$TRIBUNAL_ROOT/core/templates/verdict.md" "$PANEL_OUT/verdict.md"
  ${EDITOR} "$PANEL_OUT/verdict.md"
  echo "early stop (rule a) - verdict in $PANEL_OUT" >&2
  exit 0
fi

# --- 4. Extract each seat's DISPUTED claims VERBATIM (their own
#     sentences + EVIDENCE lines, numbered - never your paraphrase).
#     Mechanical extraction: read the R0 file, then e.g.
#         sed -n '12,19p' "$PANEL_OUT/seat-b-r0.md" > "$PANEL_OUT/disputed-from-b.md"
#     A disputed-from file looks like:
#         DISPUTED CLAIMS FROM SEAT B (verbatim):
#         B2. CLAIM: The cache layer is unnecessary because ...
#             EVIDENCE: src/cache.py:88 "..."
#         B4. CLAIM: Migration order X-then-Y corrupts ...
#             EVIDENCE: migrations/0042.sql:12 "..."
#     Pre-create the files so the editor opens real buffers and a stray quit
#     can't crash the later assembly under `set -e`; you fill them by hand.
: > "$PANEL_OUT/disputed-from-a.md"; : > "$PANEL_OUT/disputed-from-b.md"
${EDITOR} "$PANEL_OUT/disputed-from-a.md" "$PANEL_OUT/disputed-from-b.md"
#     Also extract each seat's OWN R0 claims (stateless seats need them):
: > "$PANEL_OUT/own-r0-a.md"; : > "$PANEL_OUT/own-r0-b.md"
${EDITOR} "$PANEL_OUT/own-r0-a.md" "$PANEL_OUT/own-r0-b.md"
#     If nothing was disputed, Round 1 has nothing to relay - you should have
#     taken the early stop above. Guard against an empty, meaningless Round 1:
[[ -s "$PANEL_OUT/disputed-from-a.md" || -s "$PANEL_OUT/disputed-from-b.md" ]] \
  || { echo "no disputed claims extracted - if Round 0 truly agreed, re-run and take the early stop" >&2; exit 1; }

# --- 5. Round 1: assemble mechanically - template + frozen brief +
#     own claims + opponent's disputed claims. Re-read each assembled
#     prompt before sending (verbatim-relay check).
{ cat "$TRIBUNAL_ROOT/core/templates/r1-seat.md"; echo; cat "$PANEL_OUT/frozen-brief.md"; echo; cat "$PANEL_OUT/own-r0-a.md"; echo; cat "$PANEL_OUT/disputed-from-b.md"; } > "$PANEL_OUT/r1-for-a.md"
{ cat "$TRIBUNAL_ROOT/core/templates/r1-seat.md"; echo; cat "$PANEL_OUT/frozen-brief.md"; echo; cat "$PANEL_OUT/own-r0-b.md"; echo; cat "$PANEL_OUT/disputed-from-a.md"; } > "$PANEL_OUT/r1-for-b.md"
# verbatim-relay check: eyeball each assembled prompt before it is sent - it must
# carry only the OTHER seat's DISPUTED claims (verbatim) plus this seat's own R0.
${EDITOR} "$PANEL_OUT/r1-for-a.md" "$PANEL_OUT/r1-for-b.md"
"${T[@]}" "${SEAT_A_CMD[@]}" "$(cat "$PANEL_OUT/r1-for-a.md")" > "$PANEL_OUT/seat-a-r1.md" 2> "$PANEL_OUT/seat-a-r1.err" & A_PID=$!
"${T[@]}" "${SEAT_B_CMD[@]}" "$(cat "$PANEL_OUT/r1-for-b.md")" > "$PANEL_OUT/seat-b-r1.md" 2> "$PANEL_OUT/seat-b-r1.err" & B_PID=$!
fail=0
wait "$A_PID" || { echo "seat A failed ($?)" >&2; fail=1; }
wait "$B_PID" || { echo "seat B failed ($?)" >&2; fail=1; }
(( fail == 0 )) || exit 1
[[ -s "$PANEL_OUT/seat-a-r1.md" && -s "$PANEL_OUT/seat-b-r1.md" ]] || { echo "empty seat output" >&2; exit 1; }
for f in "$PANEL_OUT"/seat-*-r1.md; do dead_seat "$f" 'ATTACK' 'ATTACK section' || exit 1; done

# --- 6. Update the ledger, run oracles on checkable residuals, then write the
#     verdict from the template. Round 2 only if a load-bearing claim was
#     overturned (core/templates/r2-revision.md).
cp "$TRIBUNAL_ROOT/core/templates/verdict.md" "$PANEL_OUT/verdict.md"
${EDITOR} "$PANEL_OUT/ledger.md" "$PANEL_OUT/verdict.md"
echo "panel complete - artifacts in $PANEL_OUT" >&2
```

Durable rules regardless of CLI:

- **cwd = artifact root** when invoking seats, so the brief's relative
  paths resolve. If a seat reports it cannot read a listed path, its
  dependent claims are `SPECULATIVE` - never relay them as `USER-FACT`.
- **Parallel & isolated Round 0** - never run seat B after reading seat
  A's output into your own context; that's a relay chain, not a panel.
- **Smoke-test, then read-test** - an arithmetic answer proves the model
  is alive; only a verbatim quote from a known artifact span proves its file
  tools reach the artifact on this host. Run both before Round 0 (script
  step 0b); fix any interceptor rather than lowering the fence.
- **Files, not ad-hoc strings** - every prompt lives in a file (audit
  trail of exactly what each seat saw).
- **Prompts contain no secrets** - they cross process boundaries and land
  in logs.

## N > 2 seats

Add more `SEAT_*_CMD` entries; Round 1 sends each seat one combined,
attributed packet of every *other* seat's disputed claims (plus its own R0
claims), and `agreed-r0` requires all seats.
